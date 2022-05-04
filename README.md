<div align="center">
  <a href="https://fineartamerica.com/profiles/jim-figora/shop">
    <img alt="eth logo" src="https://images.fineartamerica.com/images/artworkimages/mediumlarge/3/the-stoned-ape-theory-jim-figora.jpg" >
  </a>
  <p align="center">
    <a href="https://github.com/LuozhuZhang/sourceCode-zkSync-rollupContract/graphs/contributors">
      <img alt="GitHub contributors" src="https://img.shields.io/github/contributors/LuozhuZhang/Bitcoin-Implement">
    </a>
    <a href="https://GitHub.com/LuozhuZhang/Bitcoin-Implement/issues/">
      <img alt="GitHub issues" src="https://badgen.net/github/issues/LuozhuZhang/Bitcoin-Implement/">
    </a>
    <a href="http://makeapullrequest.com">
      <img alt="pull requests welcome badge" src="https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=flat">
    </a>
  </p>
</div>

This article was first published on the [here](https://mp.weixin.qq.com/s/5ps_OcxPWA96fKUn4ozsZg). I have been using gitlab before. Unfortunately, many interesting codes cannot be uploaded.

In order to better understand the internal mechanism and underlying principle of the blockchain, I tried to build a simple blockchain through Python and Flask framework

This blockchain system can mine through the workload proof algorithm, and meet the needs of transferring money between different nodes and different devices, basically realizing some simple functions of the blockchain

The following are our detailed steps: first we will build a complete blockchain framework, then we will create an API interface for this blockchain system, after the above two steps are completed, we will try to run the blockchain and create a consensus algorithm to allow it to run between different devices

# Create a blockchain

### Create the Blockchain class

We first created a Blockchain class, and in the initial function created two initialized empty lists, where chain is used to store our blocks and current_transcations is used to store transactions

```
class Blockchain(object):
    def __init__(self):
        self.chain=[]
        self.current_transactions=[]  # Store the blockchain and transactions
        
    def new_block(self):
        # Create a new block and join the blockchain network
        pass
    
    def new_transaction(self):
        # Add a new transaction
        pass
    
    @staticmethod  # @staticmethod returns a static method of a function
    def hash(block):
        # hash the last block
        pass
    
    @property  # @property turns a method into a property call
    def last_block(self):
        # Returns the last block in the blockchain
        pass
    
    def proof_of_work(self,last_proof):
        # A simple proof-of-work algorithm
        return proof
    
    @staticmethod
    def valid_proof(last_proof,proof):
        # Validation Statement
        return
```
new_block and new_transaction package new transactions into blocks, last_block and hash functions return a unique hash value for each block, proof_of_work and valid_proof form a workload proof algorithm

We will improve the specific framework of Blockchain step by step below

### What should a block look like?

Each block has an index, a Unix timestamp, a list of transactions, a checksum and the hash of the previous block

```
block={
    # 1）index
    'index':1,
    # 2）Unix timestamp
    'timestamp':1506057125.900785,
    # 3）Transaction list
    'transactions':[
        {
            'sender':"8527147fe1f5426f9dd545de4b27ee00",
            'recipient':"a77f5cdfa2934df3954a5c7c7da5df1f",
            'amount':5,
        }
    ],
    # 4）check proof
    'proof':324984774000,
    # 5）the hash of the previous block
    'previous_hash':"2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824"
}
```

The reason why each block can form a blockchain is because each new block contains the hash value of the previous block

This is why blockchains are immutable: if an attacker modifies an earlier block in the blockchain, it will change the hashes of all subsequent blocks

### Pack transactions into blocks

```
def new_transaction(self,sender,recipient,amount):
    '''
    param sender：sender's address<str>
    param recipient：recipient's address<str>
    param amount：number<int>
    return：The block index that contains this transaction<int>
    '''
        
    self.current_transactions.append({
        'sender':sender,
        'recipient':recipient,
        'amount':amount,
    })
        
    # Returns the index of the block containing the transaction
    return self.last_block['index']+1
```
The new_transaction() method adds the transaction to the transaction list and returns the index of the block containing the transaction

### Create new block

We first need to add the genesis block (the first block) to the initial function of the Blockchain, and add a proof of work for the mining result to the genesis block

```
class Blockchain(object):
    def __init__(self):
        self.current_transactions=[]
        self.chain=[]
        self.new_block(provious_hash='1',proof=100)  # Genesis Block
        
    def new_block(self,proof,previous_hash=None):
        '''
        Create a new block into the blockchain
        param proof：Proof of work generated by a proof-of-work algorithm<int>
        param previous_hash：The hash value of the previous block<str>
        return：new block<dict>
        '''
        
        block={
            'index':len(self.chain)+1,   
            'timestamp':time(),  
            'transactions':self.current_transcations,  
            'proof':proof,  
            'previous_hash':previous_hash or self.hash(self.chain[-1])  # The hash value of the previous block
        }
        
        self.current_transactions=[]  # Reset current transaction history
        self.chain.append(block)  # Add a new block to the blockchain
        return block
    
    def new_transaction(self,sender,recipent,amount):
        '''
        Create a new transaction into the next block to be mined
        param sender：sender address<str>
        param recipient：reviver address<str>
        param amount：number<int>
        return：The block index containing this transaction<int>
        '''
        
        self.current_transactions.append({
            'sender':sender,  
            'recipient':recipient,
            'amount':amount  
        })
        
        return self.last_block['index']+1  # Returns the block index of this transaction
    
    @property
    def last_block(self):
        return self.chain[-1]
    
    @staticmethod
    def hash(block):
        '''
        Generate SHA-256 value for a block
        param block：Block<dict>
        return：值<int>
        '''
        
        # We have to make sure this dictionary (block) is sorted, otherwise we will get inconsistent hashes
        block_string=json.dumps(block,sort_keys=True).encode()
        return hashlib.sha256(block_string).hexdigest()  # hexdigest() Returns the summary as a hex data string value
```

At this point, our framework is basically completed. Next, we need to understand how new blocks are created and mined

### What is Proof of Work（PoW）

A proof-of-work algorithm is used to create/mine new blocks on the blockchain: the goal of PoW is to calculate a number that meets certain conditions, and this number is computationally difficult but easy to verify

To give a simple example: Suppose the hash value of the product of an integer x times another integer y must end in 0, that is, hash(x*y)=ab23d...0

Let x=5, find y

```
from hashlib import sha256

x=5
y=0  # Because we don't yet know what y should be

while sha256(f'{x*y}'.encode()).hexdigest()[-1]!="0":
            y+=1
print(f'The solution is y={y}')
```

The result is y=21, because the generated hash value must end with 0

```
hash(5 * 21) = 1253e9373e...5e3600155e860
```

Usually the calculation difficulty is proportional to the number of specific characters that the target string needs to satisfy. Next, we implement a similar PoW algorithm: find a number P, and make the hash value of the string concatenated with the proof of the previous block equal to 4 start with 0

```
class Blockchain(object):
    ...... # omit
    
    def proof_of_work(self,last_proof):
        '''
        A simple proof-of-work algorithm:
        1) Find a number p so that hash(pp') starts with 4 0s, where p is the proof of the previous block
        2) p is the previous proof, p' is the new proof
        param last_proof：上一个proof<int>    
        return：proof<int>
        '''
        
        # Start verification from 0, if the verification is successful, return the number p, otherwise continue to verify
        proof=0
        while self.valid_proof(last_proof,proof) is False:
            proof+=1
            
        return proof
    
    @staticmethod
    def valid_proof(last_proof,proof):
        '''
        Verification statement: Does hash(last_proof, proof) contain 4 0s at the beginning?
        param last_proof：previous proof<int>
        param proof：now proof<int>
        return：Validation returns True for correctness and False for errors
        '''
        # Find the hash value of the string concatenated with the last proof of the previous block and the number proof which starts with 4 0s
        guess=f'{last_proof}{proof}'.encode()  
        guess_hash=hashlib.sha256(guess).hexdigest()  # guess_hash is a string composed of the number p and the proof of the previous block
        return guess_hash[:4]=="0000"
```

At the same time, we found that modifying the number at the beginning of 0 can modify the complexity of the algorithm, and each additional zero will greatly increase the time required for calculation

# Create API interface

We use the Python Flask framework to create three interfaces:

* /transactions/new: create a transaction and add to the block
* /mine: tell the server to mine new blocks
*/chain: returns the entire blockchain

### Create node

Our Flask server will act as a node in the blockchain network

Flask is a lightweight web application framework, suitable for small and medium-sized projects, while Django is more suitable for large-scale website development. You can learn their favorite frameworks as needed

```
import hashlib
import json
from time import time

# textwrap formats text by adjusting the position of line breaks
from textwrap import dedent
# Universally Unique Identifier: It guarantees uniqueness in space and time for all UUIDs
from uuid import uuid4
# Lightweight Web Application Framework
from flask import Flask

class Blockchain(object):
    ...... # omit
    
# instantiate our node
app=Flask(__name__)

# Generate a globally unique address for the node
node_identifier=str(uuid4()).replace('-','')   # Uuid.uuid4 generates uuid based on random number

# Instantiate the Blockchain class
blockchain=Blockchain()

# /mine interface: tell the server to mine new blocks
@app.route('/mine',methods=['GET'])
def mine():
    return "We'll mine a new Block"

# /transactions/new interface: create a transaction and add it to the block
@app.route('/transactions/new',methods=['POST'])  # Post method request, can send transaction data to the interface
def new_transaction():
    return "We'll add a new transaction"

# /chain interface: returns the entire blockchain
def full_chain():
    response={
        'chain':blockchain.chain,
        'length':len(blockchain.chain)
    }
    return jsonify(response),200

if __name__=='__main__':
    app.run(host='0.0.0.0',port=5000)  # The server runs on port 5000
```

### Send transaction

The data structure sent to the node is as follows

```
{
  'sender':'my address'
  'recipient':'someone else`s address'
  'amount':5
}
```

### The interface for sending transactions and mining

```
@app.route('/transactions/new',methods=['POST'])  # Post method request, can send transaction data to the interface
def new_transaction():
    values=request.get_json()
    
    # Check if required fields are in POST data
    required=['sender','recipient','amount']
    if not all(k in values for k in required):
        return 'Missing values',400
    
    # Create a new tx
    index=blockchain.new_transaction(values['sender'],values['recipient'],values['amount'])
    response={'message':f'Transaction will be added to Block{index}'}
    return jsonify(response),201


@app.route('/mine',methods=['GET'])  # /Mine interface: tell the server to mine new blocks
def mine():
    # We run the proof of work algorithm to get the next proof
    last_block=blockchain.last_block  # Previous block
    last_proof=last_block['proof']  # Get the proof of the previous block
    proof=blockchain.proof_of_work(last_proof) # Call proof of work
    
    # When we find the proof, we must receive the reward
    blockchain.new_transaction(
        sender='0',  # A sender of 0 indicates that this node has mined new coins
        recipient=node_identifier,  # The recipient uses this node address
        amount=1  # Reward a coin
    )
    
    # New block into chain
    previous_hash=blockchain.hash(last_block)  # Hash of previous block
    block=blockchain.new_block(proof,previous_hash)  # Add a new block to the chain
    
    # Data in this block
    response={
        'message':'New Block Forged',
        'index':block['index'],
        'transactions':block['transaction'],
        'proof':block['proof'],
        'previous_hash':block['previous_hash']
    }
    
    return jsonify(response),200
```
Note that the recipient of the transaction here is our own server node, and most of the work we do is just interacting around the Blockchain class methods

Our blockchain system cannot run across multiple devices without building a consensus algorithm

# Run the Blockchain

We choose postman as the interface debugging tool to interact with the API

First let's try mining by requesting http://localhost:5000/mine (GET):

![image](https://user-images.githubusercontent.com/70309026/166673850-bb03bf37-fd94-48fc-8d07-493f5f880204.png)

Create another transaction request:

![image](https://user-images.githubusercontent.com/70309026/166673875-b1ed05a8-d102-4bb0-a846-d3d7075c79c5.png)

# Implement a consensus algorithm

We have a basic blockchain for accepting transactions and mining, but the blockchain system should be distributed. So we need to ensure that all nodes have the same chain, and if we want to have multiple nodes on the network, we must implement a consistent algorithm

### Register node

Before implementing the consensus algorithm, we need to find a way for a node to know its neighbors. Each node needs to keep a record containing other nodes in the network, so we add the following two interfaces:

* /nodes/register: Receive a list of new nodes in URL form

* /nodes/resolve: Executes the consensus algorithm, resolves any conflicts and ensures nodes have the correct chain

Now we modify Blockchain's init function and provide a method for registering nodes

```
... # Omit Library
from urllib import urlparse
...

class Blockchain(object):
    def __init__(self):
        ... # Omit other functions
        self.nodes=set()  # Storage node
        ...
        
    def register_node(self,address):
        '''
        Add a new node to the node list
        param address：Node address, for examplehttp://192.168.0.5:5000<str> 
        return：None
        '''
        
        parsed_url=urlparse(address)
        self.nodes.add(parsed_url.netloc)  # Store node address in set
```
We use a set (set) to store nodes, which is an easy way to avoid adding nodes repeatedly

### Consensus algorithm

Conflicts arise when one node has a different chain from another node. To solve this problem, we stipulate that the longest valid chain is the most authoritative. We will use this algorithm to reach consensus among the nodes in the network

```
......  # Omit Library
import requests

class Blockchain(object):
    ......  # Omit other functions
    
    def valid_chain(self,chain):  # Check whether each chain is valid by traversing each block and verifying the hash and proof
        '''
        确定给定的区块链是否有效      
        param chain：<list>A blockchain    
        return ：<bool>True valid, false invalid
        '''
        last_block=chain[0]  # Start with the first one
        current_index=1
        
        while current_index<len(chain):  # Traverse all blocks
            block=chain[current_index]
            print(f'{last_block}')
            print(f'{block}')
            print(f'\n----------\n')
            # Check whether the hash of the block is correct
            if block['previous_hash']!=self.hash(last_block):
                return False
            
            #  Check whether Po W is correct
            if not self.valid_proof(last_block['proof'],block['proof']):
                return False
            
            last_block=block
            current_index+=1
            
        return True
    
    def resolve_conflicts(self):  # Traverse all our neighbor nodes, download their chains and use the above method to verify them. If we find a longer chain, replace our chain
        '''
        This is our consensus algorithm: it solves conflicts by replacing our chain with the longest chain in the network 
        return：<bool> If our chain is replaced, it returns true; otherwise, it returns false
        '''
        neighbours=self.nodes # Neighbor node
        new_chain=None
        
        # Just look for a chain longer than ours
        max_lengeth=len(self.chain)  # The length of our chain
        
        # Grab and validate chains from all nodes in our network
        for node in neighbours:
            response=requests.get(f'http://{node}/chain')
            
            if response.status_code=200:
                length=response.json()['length']
                chain=response.json()['chain']
                
                # Check if the length is longer
                if length>max_length and self.valid_chain(chain):  # 比我们链更长的有效链：替换新长度和新链
                    max_length=length
                    new_chain=chain
                    
        # Replace our chain if the new chain is longer than ours
        if new_chain:
            self.chain=new_chain
            return True
        
        return False
```
Register two endpoints with our API: one for adding adjacent nodes and one for resolving conflicts

```
# Add adjacent nodes
@app.route('/nodes/register',methods=['POST'])
def register_nodes():
    values=request.get_json()
    
    nodes=values.get('nodes')
    if nodes is None:
        return 'Error:Please supply a valid list of nodes',400
    
    for node in nodes:
        blockchain.register_node(node)
        
    response={
        'message':'New nodes have been added',
        'total_nodes':list(blockchain.nodes)
    }
    return jsonify(response),201

# Consensus Algorithm: Determining the Longest Chain
@app.route('/nodes/resolve',methods=['GET'])
def consensus():
    replaced=blockchain.resolve_conflicts()
    
    if replaced:
        response={
            'message':'Our chain was replaced',
            'new_chain':blockchain.chain
        }
    else:
        response={
            'message':'Our chain is authoritative',
            'chain':blockchain.chain
        }
        
    return jsonify(response),200
```
Now we can use different machines to start different nodes (or different ports of a machine) on our network, which means we can use the blockchain with our friends

We can choose different nodes of one machine for transfer testing, or use multiple different machines:

![image](https://user-images.githubusercontent.com/70309026/166674178-f15492fc-a331-45e0-8853-7ce7840097df.png)

---

# 中文

为了更好的理解区块链的内部机制和底层原理，我尝试通过Python和Flask框架构建了一个简单的区块链

这个区块链系统可以通过工作量证明算法进行挖矿，以及满足在不同节点和不同设备之间转账的需要，基本实现了区块链的一些简单功能

以下是我们的详细步骤：首先我们会构建一个完整的区块链框架，之后我们会为这个区块链系统打造API接口，在以上两步完成以后我们会尝试运行区块链，并打造共识算法以允许它在不同设备之间运行

# 创建区块链

### 创建Blockchain类

我们首先创建了一个Blockchain类，并且在初始函数中创建了两个初始化的空列表，其中chain用于存储我们的区块，current_transcations用于存储交易

```
class Blockchain(object):
    def __init__(self):
        self.chain=[]
        self.current_transactions=[]  # 存储区块链和交易
        
    def new_block(self):
        # 创建一个新区块并加入区块链网络中
        pass
    
    def new_transaction(self):
        # 增加一笔新交易
        pass
    
    @staticmethod  # @staticmethod返回函数的静态方法
    def hash(block):
        # hash一个区块
        pass
    
    @property  # @property把一个方法变成属性调用的
    def last_block(self):
        # 返回区块链中最后一个区块
        pass
    
    def proof_of_work(self,last_proof):
        # 一个简单的工作量证明算法：
        return proof
    
    @staticmethod
    def valid_proof(last_proof,proof):
        # 验证声明
        return
```
new_block和new_transaction将新交易打包入区块中，last_block和hash函数给每个区块返回一个唯一的哈希值，proof_of_work和valid_proof组成工作量证明算法

我们会在下文中一步步完善Blockchain的具体框架

### 一个区块长什么样？

每个区块都有一个索引，一个Unix时间戳（timestamp），一个事务列表，一个校验和前一个区块的哈希值

```
block={
    # 1）索引
    'index':1,
    # 2）Unix时间戳
    'timestamp':1506057125.900785,
    # 3）事务列表
    'transactions':[
        {
            'sender':"8527147fe1f5426f9dd545de4b27ee00",
            'recipient':"a77f5cdfa2934df3954a5c7c7da5df1f",
            'amount':5,
        }
    ],
    # 4）校验
    'proof':324984774000,
    # 5）前一个块的散列
    'previous_hash':"2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824"
}
```

每个区块之所以能够组成区块链，正是因为每个新的区块都包含前一个区块的哈希值

这便是区块链不可更改的原因：如果攻击者修改了区块链中较早的区块，则会改变接下来所有区块的哈希值

### 将交易打包入区块

```
def new_transaction(self,sender,recipient,amount):
    '''
    param sender：发送者的地址<str>
    param recipient：接收者的地址<str>
    param amount：数量<int>
    return：包含这笔交易的区块索引<int>
    '''
        
    self.current_transactions.append({
        'sender':sender,
        'recipient':recipient,
        'amount':amount,
    })
        
    # 返回包含该交易的区块索引
    return self.last_block['index']+1
```
new_transaction()方法将交易添加入了交易列表中，并且返回包含该交易的区块的索引

### 创建新的区块

我们首先需要在Blockchain的初始函数中添加创世区块（第一个区块），并且要为起源块添加一个挖矿结果的工作证明

```
class Blockchain(object):
    def __init__(self):
        self.current_transactions=[]
        self.chain=[]
        self.new_block(provious_hash='1',proof=100)  # 创世区块
        
    def new_block(self,proof,previous_hash=None):
        '''
        创建一个新的区块到区块链中
        param proof：工作证明算法生成的证明<int>
        param previous_hash：前一个区块的hash值<str>
        return：新区块<dict>
        '''
        
        block={
            'index':len(self.chain)+1,   
            'timestamp':time(),  
            'transactions':self.current_transcations,  
            'proof':proof,  
            'previous_hash':previous_hash or self.hash(self.chain[-1])  # 前一个区块的hash值
        }
        
        self.current_transactions=[]  # 重置当前交易记录
        self.chain.append(block)  # 将新区块加入区块链中
        return block
    
    def new_transaction(self,sender,recipent,amount):
        '''
        创建一笔新交易到下一个被挖掘的区块中
        param sender：发送人地址<str>
        param recipient：接收人地址<str>
        param amount：数量<int>
        return：包含本次交易的区块索引<int>
        '''
        
        self.current_transactions.append({
            'sender':sender,  
            'recipient':recipient,
            'amount':amount  
        })
        
        return self.last_block['index']+1  # 返回本次交易的区块索引
    
    @property
    def last_block(self):
        return self.chain[-1]
    
    @staticmethod
    def hash(block):
        '''
        给一个区块生成SHA-256值
        param block：Block<dict>
        return：值<int>
        '''
        
        # 我们必须确保这个字典（区块）是经过排序的，否则我们将会得到不一致的散列
        block_string=json.dumps(block,sort_keys=True).encode()
        return hashlib.sha256(block_string).hexdigest()  # hexdigest()返回摘要，作为十六进制数据字符串值
```
到此我们的框架基本搭建完毕，接下来我们需要了解新的区块是怎么被创建挖掘的

### 什么是工作量证明算法（PoW）

工作量证明算法用于在区块链上创建/挖掘新的区块：PoW的目标是计算出一个符合特定条件的数字，并且这个数字虽然计算困难但是易于验证

举个简单的例子：假设一个整数x乘以另一个整数y的积的Hash值必须以0结尾，即hash(x*y)=ab23d……0

设x=5，求y

```
from hashlib import sha256

x=5
y=0  # 因为我们还不知道y应该是多少

while sha256(f'{x*y}'.encode()).hexdigest()[-1]!="0":
            y+=1
print(f'The solution is y={y}')
```
结果是y=21，因为生成的Hash值结尾必须为0


```
hash(5 * 21) = 1253e9373e...5e3600155e860
```

通常计算难度与目标字符串需要满足的特定字符数量成正比，接下来我们实现一个相似PoW算法：找到一个数字P，使它与前一个区块的proof拼接成的字符串的Hash值以4个0开头

```
class Blockchain(object):
    ...... # 省略
    
    def proof_of_work(self,last_proof):
        '''
        一个简单的工作量证明算法：
        1）找到一个数字p，使hash(pp')以4个0开头，其中p是前一个区块的proof
        2）p是之前的proof，p'是新的proof    
        param last_proof：上一个proof<int>    
        return：proof<int>
        '''
        
        # 从0开始验证，如果验证成功返回数字p，否则继续验证
        proof=0
        while self.valid_proof(last_proof,proof) is False:
            proof+=1
            
        return proof
    
    @staticmethod
    def valid_proof(last_proof,proof):
        '''
        验证声明：hash(last_proof,proof)开头是否包含4个0？
        param last_proof：之前的proof<int>
        param proof：现在的proof<int>
        return：验证正确返回True，错误返回False
        '''
        
        guess=f'{last_proof}{proof}'.encode()  # 找到数字proof which 与前一个区块的last proof拼接成的字符串的Hash值以4个0开头
        guess_hash=hashlib.sha256(guess).hexdigest()  # guess_hash是数字p与上一个区块的proof拼接成的字符串
        return guess_hash[:4]=="0000"
```

同时我们发现修改0开头的个数可以修改算法的复杂度，每多一个零都会大大增加计算所需要的时间

# 创建API接口

我们使用Python Flask框架分别创建三个接口：

* /transactions/new：创建一个交易并添加到区块
* /mine：告诉服务器去挖掘新的区块
* /chain：返回整个区块链

### 创建节点

我们的Flask服务器将扮演区块链网络中的一个节点

Flask是一个轻量Web应用框架，适用于中小型项目，而Django更适用于大型网站开发，小伙伴们可以按照需要学习自己喜欢的框架

```
import hashlib
import json
from time import time

# textwrap通过调整换行符的位置来格式化文本
from textwrap import dedent
# 通用唯一标识：对于所有的UUID它可以保证在空间和时间上的唯一性
from uuid import uuid4
# 轻量级 Web 应用框架
from flask import Flask

class Blockchain(object):
    ...... # 省略
    
# 实例化我们的节点
app=Flask(__name__)

# 为该节点生成一个全局唯一的地址
node_identifier=str(uuid4()).replace('-','')   # uuid.uuid4基于随机数来生成UUID

# 实例化Blockchain类
blockchain=Blockchain()

# /mine接口：告诉服务器去挖掘新的区块
@app.route('/mine',methods=['GET'])
def mine():
    return "We'll mine a new Block"

# /transactions/new接口：创建一个交易并添加到区块
@app.route('/transactions/new',methods=['POST'])  # POST方式请求，可以给接口发送交易数据
def new_transaction():
    return "We'll add a new transaction"

# /chain接口：返回整个区块链
def full_chain():
    response={
        'chain':blockchain.chain,
        'length':len(blockchain.chain)
    }
    return jsonify(response),200

if __name__=='__main__':
    app.run(host='0.0.0.0',port=5000)  # 服务器运行端口5000
```

### 发送交易
发送到节点的数据结构如下
```
{
  'sender':'my address'
  'recipient':'someone else`s address'
  'amount':5
}
```

### 发送交易与挖矿的接口

```
@app.route('/transactions/new',methods=['POST'])  # POST方式请求，可以给接口发送交易数据
def new_transaction():
    values=request.get_json()
    
    # 检查必填字段是否在POST数据中
    required=['sender','recipient','amount']
    if not all(k in values for k in required):
        return 'Missing values',400
    
    # 创建一笔新交易
    index=blockchain.new_transaction(values['sender'],values['recipient'],values['amount'])
    response={'message':f'Transaction will be added to Block{index}'}
    return jsonify(response),201


@app.route('/mine',methods=['GET'])  # /mine接口：告诉服务器去挖掘新的区块
def mine():
    # 我们运行工作证明算法以获取下一个证明
    last_block=blockchain.last_block  # 上一个区块
    last_proof=last_block['proof']  # 获取上一个区块的证明
    proof=blockchain.proof_of_work(last_proof) # 调用proof_of_work
    
    # 找到证明以后我们必须收到奖励
    blockchain.new_transaction(
        sender='0',  # 发件人为0表示此节点已开采新硬币
        recipient=node_identifier,  # 收件人使用本节点地址
        amount=1  # 奖励一枚硬币
    )
    
    # 新块入链
    previous_hash=blockchain.hash(last_block)  # 上一个区块的hash
    block=blockchain.new_block(proof,previous_hash)  # 将新块添加入链中
    
    # 本区块数据
    response={
        'message':'New Block Forged',
        'index':block['index'],
        'transactions':block['transaction'],
        'proof':block['proof'],
        'previous_hash':block['previous_hash']
    }
    
    return jsonify(response),200
```
注意，这里的交易的接收者是我们自己的服务器节点，我们做的大部分工作都只是围绕Blockchain类方法进行交互

在没有打造共识算法之前，我们的区块链系统还不能在多台设备之间运行

# 运行区块链

我们选择postman作为接口调试工具，去与API进行交互

首先让我们通过请求http://localhost:5000/mine（GET）来尝试挖矿：

![image](https://user-images.githubusercontent.com/70309026/166673850-bb03bf37-fd94-48fc-8d07-493f5f880204.png)

再创建一个交易请求：

![image](https://user-images.githubusercontent.com/70309026/166673875-b1ed05a8-d102-4bb0-a846-d3d7075c79c5.png)

# 实现共识算法

我们有了一个基本的区块链可用于接受交易和挖矿，但区块链系统应该是分布式的。所以我们就需要保证所有节点有同样的链，而我们若想在网络上有多个节点，就必须实现一个一致性的算法

### 注册节点

在实现一致性算法之前，我们需要找到一种方式让一个节点知道它相邻的节点。每个节点都需要保存一份包含网络中其他节点的记录，所以我们新增以下两个接口：

* /nodes/register：接收URL形式的新节点列表

* /nodes/resolve：执行一致性算法，解决任何冲突，确保节点拥有正确的链

现在我们修改Blockchain的init函数，并且提供一个注册节点的方法

```
... # 省略库
from urllib import urlparse
...

class Blockchain(object):
    def __init__(self):
        ... # 省略其他函数
        self.nodes=set()  # 存储节点
        ...
        
    def register_node(self,address):
        '''
        往节点列表中增加一个新节点 
        param address：节点地址，例如http://192.168.0.5:5000<str> 
        return：None
        '''
        
        parsed_url=urlparse(address)
        self.nodes.add(parsed_url.netloc)  # 把节点地址存入set中
```
我们用set(集合)来存储节点，这是一种避免重复添加节点的简单方法

### 共识算法

当一个节点与另一个节点有不同的链时就会产生冲突。为了解决这个问题，我们规定最长的有效链是最权威的。我们将使用这个算法，在网络中的节点之间中达成共识
```
......  # 省略库
import requests

class Blockchain(object):
    ......  # 省略其他函数
    
    def valid_chain(self,chain):  # 通过遍历每个块并验证散列和证明来检查每一个链是否有效
        '''
        确定给定的区块链是否有效      
        param chain：<list>一个区块链      
        return ：<bool>True有效、Flase无效
        '''
        last_block=chain[0]  # 从第一个开始
        current_index=1
        
        while current_index<len(chain):  # 遍历所有区块
            block=chain[current_index]
            print(f'{last_block}')
            print(f'{block}')
            print(f'\n----------\n')
            # 检查区块的hash是否正确
            if block['previous_hash']!=self.hash(last_block):
                return False
            
            #  检查PoW是否正确
            if not self.valid_proof(last_block['proof'],block['proof']):
                return False
            
            last_block=block
            current_index+=1
            
        return True
    
    def resolve_conflicts(self):  # 遍历我们所有邻居节点，下载它们的链并使用上面的方法验证它们，如果找到一个更长链就替换我们的链
        '''
        这是我们的共识算法：它通过网络中最长的链替换我们的链来解决冲突     
        return：<bool> 如果我们的链被替换了返回True，否则返回False
        '''
        neighbours=self.nodes # 邻居节点
        new_chain=None
        
        # 只需要寻找比我们的链更长的链
        max_lengeth=len(self.chain)  # 我们链的长度
        
        # 抓取并验证来自我们网络中所有节点的链
        for node in neighbours:
            response=requests.get(f'http://{node}/chain')
            
            if response.status_code=200:
                length=response.json()['length']
                chain=response.json()['chain']
                
                # 检查长度是否更长
                if length>max_length and self.valid_chain(chain):  # 比我们链更长的有效链：替换新长度和新链
                    max_length=length
                    new_chain=chain
                    
        # 如果新链比我们的链更长，替换我们的链
        if new_chain:
            self.chain=new_chain
            return True
        
        return False
```
将两个端点注册到我们的API中：一个用于添加相邻节点，另一个用于解决冲突
```
# 添加相邻节点
@app.route('/nodes/register',methods=['POST'])
def register_nodes():
    values=request.get_json()
    
    nodes=values.get('nodes')
    if nodes is None:
        return 'Error:Please supply a valid list of nodes',400
    
    for node in nodes:
        blockchain.register_node(node)
        
    response={
        'message':'New nodes have been added',
        'total_nodes':list(blockchain.nodes)
    }
    return jsonify(response),201

# 共识算法：确定最长链
@app.route('/nodes/resolve',methods=['GET'])
def consensus():
    replaced=blockchain.resolve_conflicts()
    
    if replaced:
        response={
            'message':'Our chain was replaced',
            'new_chain':blockchain.chain
        }
    else:
        response={
            'message':'Our chain is authoritative',
            'chain':blockchain.chain
        }
        
    return jsonify(response),200
```
现在我们可以使用不同的机器在我们的网络上启动不同的节点（或一台机器的不同端口），也就是说我们可以和身边的小伙伴一起使用这个区块链了

我们可以选择一台机器的不同节点进行转账测试，或者使用多台不同的机器：

![image](https://user-images.githubusercontent.com/70309026/166674178-f15492fc-a331-45e0-8853-7ce7840097df.png)
