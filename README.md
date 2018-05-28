# Bug Bounty Program for the Mokens Contract

This bug bounty program applies to the [Mokens contract](Mokens.sol) in this repository.

I am willing to pay up to 1 ETH total for security vulnerabilities or other bugs found or for important gas optimizations. The amount of ETH paid for a bug found will depend on the severity and importance of the bug and the helpfullness of the bug report. No ETH will be paid for bugs that have been previously reported.

To report a bug create an issue in this repository about it.

Generally, any useful comments, about bugs or not, are appreciated.

The bug bounty starts now and ends on 30 May 2018 or potentially later than that.

To understand what mokens are about read the webpage about mokens here: https://mokens.io/about

Currently the beta version of the mokens website is running here: https://mokens.io/

You can chat with me and ask question in the [Mokens Discord Channel](https://discord.gg/ZyaqFhE).

## Understanding the Mokens Contract

The mokens contract implements the ERC721, ERC721Enumerable, ERC721Metadata and ERC165 interfaces.

The contract mints ERC721-based crypto-collectibles called "mokens".

The contract has been gas-optimized. Specifically the mint function has been optimized to require as little gas as possible while still implementing the needed functionality.

### The tokenId

Each moken has a tokenId that identifies it. tokenIds start at 0 and increment. The contract contains a list of all mokens via the `mapping (uint256 => Moken) private mokens;` mapping. The position of each moken in the list/mapping is the same as its tokenId. This makes the implementation of the `tokenByIndex(uint256 _index)` function from the ERC721Enumerable interface very easy to implement:
```  
function tokenByIndex(uint256 _index) external view returns (uint256 tokenId) {
    require(_index < mokensLength, "TokenId at this index does not exist.");
    return _index;
}
```
And it makes retrieving a moken by index or by tokenId the same thing since the index and tokenId are the same for each moken. It also means that mokens cannot be deleted because that would change the indexes of mokens in the list of all mokens.

### era

An 'Era' is a set of mokens that are created in a span of time. The first and current era is "Genesis". Newly minted mokens are Genesis mokens. When the next era starts no more Genesis mokens will be minted. [Read more about it from the about webpage.](https://mokens.io/about/eras)

The era in which a moken is created is stored with the moken. 

### _linkData

The mint function has a bytes32 _linkHash argument. _linkHash contains a hash of data contents to be associated/linked with the moken that is minted. This is a way to associate/connect off-chain data with a moken. For example the mokens.io website makes a keccak256 hash of the moken description, moken image bytes and moken attributes and passes that hash into the mint function of the mokens contract.

### Moken Data

Each moken stores an instance of a Moken struct. The Moken struct consists of a moken name and a 32 byte storage slot called "data". The "data" storage variable contains the first 8 bytes from the _linkHash variable, a 2 byte unsigned integer that is an index into the list of all eras which is used to get the era for the moken, a two byte unsigned integer that is the position of the moken in the list of mokens owned by the owner address, and the address of the owner of the moken. Storing all these data in one 256 bit storage slot saves gas when minting, and saves gas when reading this data from storage.

### Mokens

Within the scope of the Mokens Contract a moken consists of a tokenId, a name, a lowercase version of the name, an era, an owner address, an index position in the list of owner mokens, an 8-byte hash of data contents and a tokenURI.

### Burning/Deleting Mokens

You will notice that there is no burn/delete moken function in the contract. This functionality was left out because this functionality would require a mapping from tokenId to index in the `mapping (uint256 => Moken) private mokens;` mapping which would add at least 20,000 more gas to the mint function and add gas other places. Keeping the tokenId the same as its index position in the list of all mokens reduces gas and simplifies the implementation of functions such as tokenByIndex. 

I think most users will not want to delete/burn their mokens and when they do they can get rid of them by selling them or sending them to another ethereum address.

### Mint Price

The price to mint a moken is determined by a base price plus the number of tokens times a price: `uint256 currentMintPrice = mintBasePrice + (tokenId * mintStepPrice_);`

Since the next tokenId increments each time a mint occurs the price for mokens increases each time a mint occurs. A race condition is possible when multiple people try to mint a moken at the same time. If multiple people try to mint a moken at the same time one of the mint transactions will go through and the others will throw because the mint price went up. To prevent these transactions from throwing a mint price buffer exists. The mint price buffer enables mint transactions to go through if they are within the buffer range. It is noted here that people are supposed to mint mokens at the current price as given from the `mintPrice()` function. It is possible for morally corrupt people to get a slight discount by paying a little less than the price given from the `mintPrice()` function, however they will get no discount from their conscience if they have one, and karma will eventually get them if they don't.




