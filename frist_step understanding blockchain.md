
# 认识区块链的链原理
# 一、什么是链？<br>
　　如果你是一个程序猿，或者说是IT狗，或者说你是你接触使用过github，gitlab等仓库的话，那么你对分布式的概念一定是有一定的理解的，而且你对分支以及树形
也是有一定理解的。回归到区块链的书面术语：区块链是分布的记账账本。<br>
    从GITHUB的理解来说，我们每一个程序员都是有一个仓库，也就是区块链中的分布式数据库，我们所有的修改更新等记录在本地都有完整的数据记录。只是当大家修
改完成时通过push或者pull将每一个开发者的代码进行合并进而组装成一个完整的项目。那么问题来了，有的人会问，这样讲的话，GITHUB网站不就是一个最大的中心服
务器嘛？在这里我想告诉大家的是GITHUB网站只不过是为了方便大家有一个共同的互联网组织进行操作而已。在现实的工作环境中，如果我们没有互联网环境，只有局域
网环境，我们一样可以使用GIT去进行工作，我可以pull张三的库进行代码的修改，同时也可以push给张三，李四同样也可以pull我的代码进行修改，当然也可以将修改
后的代码push给我。那么从这里来看是不是跟区块链有一点点类似呢？<br>
　　从这里我们只是理解了什么是分布式，什么是链？关于什么是链，我通过golang的代码通俗易懂的给大家讲解一下。当然这其中我也借鉴了许多前辈的东西，希望大
家可以共同学习。<br>
# 二、什么是区块<br>
　　区块可以理解为每一个数据记录的步骤，类似GIT当中的每一步提交。有的朋友会问为什么不是每一次修改呢？当你的每一次修改并没有与GIT产生关联时，他又怎么
会知道你的操作呢？<br>
　　在比特币等加密货币中，每一个区块存储的是交易信息，其实也就是转账记录，后面我会讲到，早期的加密贷币为了更安全并没有存储余额、用户等信息，全是交易
记录。交易记录也是加密货币的根本。在GIT操作中，我们可以理解的每个区块存储的是我们提交时对每个文件的操作。<br>
    无论是存储哪一类信息，存储什么信息，我们均可以以这样的原型去构建一下它的数据结构：<br>
type Block struct {<br>
    Timestamp     int64<br>
    Data          []byte<br>
    PrevBlockHash []byte<br>
    Hash          []byte<br>
}<br>
　　Timestamp 是当前时间戳，也就是区块创建的时间。<br>
　　Data 是区块存储的实际有效的信息。
　　PrevBlockHash 存储的是前一个块的哈希。<br>
　　Hash 是当前块的哈希。<br>
    当然，如果你喜欢，你也可以在这个block区块中再去增加更多的数据项。这里我们只是简单的举例，其实看到这个数据结构时，很多朋友估计就非常明白了吧？这是
不是就是一个典型的树形结构呢？是的。<br>
    如何生成hash呢，我这里我简单的举一个例子，参照别人的案例，将（Timestamp, Data 和 PrevBlockHash）进行连接，然后连接后的结果，我们通过一个sha-
256进行运算即可以得到一个当前的hash值。<br>
   func (b *Block) SetHash() {<br>
    timestamp := []byte(strconv.FormatInt(b.Timestamp, 10))<br>
    headers := bytes.Join([][]byte{b.PrevBlockHash, b.Data, timestamp}, []byte{})<br>
    hash := sha256.Sum256(headers)<br>

    b.Hash = hash[:]
   }<br>
   接下来，我们来创建一个区块<br>
   func NewBlock(data string, prevBlockHash []byte) *Block {<br>
     block := &Block{time.Now().Unix(), []byte(data), prevBlockHash, []byte{}}<br>
     block.SetHash()<br>
     return block<br>
  }<br>
   到这里我们的区块就已经完成了。<br>
# 三、区块链<br>
　　接下来，我们来看一下，如果创建一个区块链。区块链从本质上来说，就是所有的区块通过前一个hash与后一个hash进行有序排列。<br>
    我们来创建一个区块链的数据结构。<br>
    type blockchain struct{<br>
       blocks *[]Block <br>
   }<br>
   接下来，我们来写一个为区块链添加区块的函数：<br>
    func (bc *blockchain) Addblock(data string){<br>
	//data,表示的是数据内容，我们更多的应该去以json的方式进行存储<br>
	prevblock := bc.blocks[len(bc.blocks)-1]//查找当前区块链上的最后一个块，为什么要减1呢？因为golang是以0作为起始下标，另外有一种golang下
标是以1作为起始，一下忘记了，后面补上<br>
       newblock := NewBlock(data,prevblock)//创建一个新的区块链<br>
       bc.blocks = append(bc.blocks,newblock)//将当前的链上连接上新的区块<br>
    }<br>
    基本的内容我们已经完成了，那么接上来我们要创造世界上的第一个区块，既币圈所谓的创始区块：<br>
    func Newfristblock(data string) *Block {<br>
        return NewBlock(data,[]byte{})//第一个区块链的上一个hash为空 <br>
    }<br>
现在，我们可以实现一个函数来创建有创世块的区块链：<br>
<br>
func NewBlockchain() *Blockchain {<br>
    return &Blockchain{[]*Block{Newfristblock("This is frist blockchain")}}<br>
}<br>
来检查一个我们的区块链是否如期工作：<br>

func main() {<br>
    bc := NewBlockchain()<br>

    bc.AddBlock("我将我的1000万个比特币转给了中本聪")<br>
    bc.AddBlock("本中聪将1000万个比特币，转给了王思聪，于是它就是世界上最富有的人")<br>

    for _, block := range bc.blocks {
        fmt.Printf("Prev. hash: %x\n", block.PrevBlockHash)
        fmt.Printf("Data: %s\n", block.Data)
        fmt.Printf("Hash: %x\n", block.Hash)
        fmt.Println()
    }
}
<br>
至于输出结果，有兴趣的朋友按照我的方式，进行去创建一个main.go文件进行运行，就可以看到一条简单的区块链形成。<br>

区块链的概念比较容易理解，但是数字货币的关键还有共识算法、智能合约等技术，我将会跟大家一起学习相关技术，希望可以共同进步。<br>
