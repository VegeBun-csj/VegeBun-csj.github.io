---
title: "交易invoke的使用"
date: 2020-08-29T14:25:47+08:00
tags:
- 区块链
Categories:
- Hyperledger Fabric--NodeSDK开发
---



## Fabric 1.4.x的新版高级API(Fabrci-NetWork模块)的使用

> 新版api引入了fabric-network模块，在原来的fabric-client模块的基础上那个进行了再封装，对于交易的提交incvoke,查询query大大简化，该模块最核心的是两个功能类:
>
> 1.`Gateway`（网关类）:负责连接网络，并获取网络network实例，从而获取相应channel实例，合约contract实例，从而进行交易，以及相关事件的监听
>
> 2.`FileSystemWallet`（身份文件类）：创建并保存客户端的身份信息，用来进行网络通信的依据

老版本的api(1.4.x之前的版本)中提供了client创建transactionId的相关方法，并能返回相应的transactionId,但是对于官方的fabcar的案例，其中的invoke交易的使用非常的简单，但是却找不到相关的获取transactionId的方法,下面一行代码即可搞定invoke类型的交易，参数是（链码中的方法名，args[方法参数]），封装的极为“严实”

```javascript
        await contract.submitTransaction('createCar', 'CAR12', 'Honda', 'Accord', 'Black', 'Tom');
```

看这行代码时，找不到如何获取交易的ID,所以来详细看一下` submitTransaction` 方法（该方法分别做了两件事：通过链码方法名创建transaction对象，再通过transaction对象的submit方法将链码方法的参数传入完成交易的背书），该方法位于`contract` 的模块中

```javascript
async submitTransaction(name, ...args) {
		return this.createTransaction(name).submit(...args);
	}
```

在contract文件中再看到 `createTransaction` 方法

```javascript
    /* @param {String} name Transaction function name.
	 * @returns {module:fabric-network.Transaction} A transaction object.
			 返回的是transaction对象
     */
	createTransaction(name) {
		verifyTransactionName(name);
		const qualifiedName = this._getQualifiedName(name);
		const transaction = new Transaction(this, qualifiedName);
		const eventHandlerStrategy = this.getEventHandlerOptions().strategy;
		if (eventHandlerStrategy) {
			transaction.setEventHandlerStrategy(eventHandlerStrategy);
		}
		return transaction;
	}
```

此时转到 `transaction` 文件中看到 `submit(...args)` 方法

```javascript
	/**
	 * Submit a transaction to the ledger. The transaction function <code>name</code>
	 * will be evaluated on the endorsing peers and then submitted to the ordering service
	 * for committing to the ledger.
	 * @async
     * @param {...String} [args] Transaction function arguments.   参数是调用的链码方法的所需参数
     * @returns {Buffer} Payload response from the transaction function.
	 * @throws {module:fabric-network.TimeoutError} If the transaction was successfully submitted to the orderer but
	 * timed out before a commit event was received from peers.
     */
	async submit(...args) {
		verifyArguments(args);
		this._setInvokedOrThrow();

		const network = this._contract.getNetwork();
		const channel = network.getChannel();
		const txId = this._transactionId.getTransactionID();
		const options = this._contract.getEventHandlerOptions();
		const eventHandler = this._createTxEventHandler(this, network, options);

		const request = this._buildRequest(args);
		if (this._endorsingPeers) {
			request.targets = this._endorsingPeers;
		}

		const commitTimeout = options.commitTimeout * 1000; // in ms
		let timeout = this._contract.gateway.getClient().getConfigSetting('request-timeout', commitTimeout);
		if (timeout < commitTimeout) {
			timeout = commitTimeout;
		}

		// node sdk will target all peers on the channel that are endorsingPeer or do something special for a discovery environment
		const results = await channel.sendTransactionProposal(request, timeout);
		const proposalResponses = results[0];
		const proposal = results[1];

		// get only the valid responses to submit to the orderer
		const {validResponses} = this._validatePeerResponses(proposalResponses);

		await eventHandler.startListening();

		// Submit the endorsed transaction to the primary orderers.
		const response = await channel.sendTransaction({
			proposalResponses: validResponses,
			proposal
		});

		if (response.status !== 'SUCCESS') {
			const msg = util.format('Failed to send peer responses for transaction %j to orderer. Response status: %j', txId, response.status);
			logger.error('submit:', msg);
			eventHandler.cancelListening();
			throw new Error(msg);
		}

		await eventHandler.waitForEvents();

		return validResponses[0].response.payload || null;
	}
```



通过上面的分析，我们能可以不使用`submitTransaction` 方法 ，而是使用其中的拆解的方法，分两步，从而提取出其中的tx_id ,在**Fabric-network-Transaction类中看到有getTransactionID()方法,但是该方法返回的是一个transactionID对象，transactionId对象还有一个getTransationID方法，从而获得tx_id**

```javascript
 		const transaction = contract.createTransaction('createCar');
        await transaction.submit('CAR21', 'hahaha', 'cccc', 'black', 'cccccc');
        const tx_id = transaction.getTransactionID().getTransactionID();
```

> 这种写法第1，2行将原来的submitTransaction方法拆解为两步骤，由第一步的返回transaction对象来取得tx_id

在获得了tx_id之后，有很多方面都可以顺利进行，包括channel中根据交易ID来进行相关的查询操作，包括事件的监听，可以通过再提交交易之后，立即返回交易id，从而避免了多余的查询。

