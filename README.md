# IDB3 LIBRARY
Library for reading IDA Pro databases.

## About

The header file `idb3.h` contains a library for reading from IDA Pro databases.

## File format

[File format description](IDB-FORMAT.md)

## Library

### Types

#### IDBFile

Class for accessing sections of an `.idb` or `.i64` file.

Constructor Parameters:

 * `std::shared_ptr<std::istream>` ( typedefed to `stream_ptr` )

Methods:

 * `stream_ptr getsection(int)`

#### ID0File, ID1File, NAMFile

Constructor Parameters:

 * `IDBFile& idb`
 * `stream_ptr`

Constant

 * `INDEX`  - the argument for `idb.getsection`

#### ID0File

Methods

 * `Cursor find(relation_t, nodeid, ...)` 
	* `...`  can be: 
		* tag, index
		* tag, hash
		* tag
 * `Cursor find(relation_t, std::string key)`
 * `std::string blob(nodeid, tag, ...)`
 * `uint64_t node(std::string name)`

 * `bool is64bit()`
	* `true` for `.i64` files.

 * `uint64_t nodebase()`
	* return `0xFF000000(00000000)` for 32/64 bit databases.

 * `void enumlist(uint64_t nodeid, char tag, CB cb)`
	* call `cb` for each value in the list.

Convenience Methods

 * `std::string getdata(ARGS...args)`
 * `std::string getstr(ARGS...args)`
 * `uint64_t getuint(ARGS...args)`
 * `uint64_t getuint(BtreeBase::Cursor& c)`
 * `std::string getname(uint64_t node)`

#### ID1File

Methods

 * `uint32_t GetFlags(uint64_t ea)`


#### NAMFile

Methods

 * `uint64_t findname(uint64_t ea)`


#### Cursor

Methods

 * `void next()`
	* move cursor to the next btree record
 * `void prev()`
	* move cursor to the previous btree record
 * `bool eof()`
	* did we reach the start/end of the btree?
 * `std::string getkey()`
	* return the key pointed to by the cursor
 * `std::string getval()`
	* return the value pointed to by the cursor

### Options

Define `IDB_ZLIB_COMPRESSION_SUPPORT` to enable support compressed databases (requires `zlib` library).

### Example

```c++
#include <idb3.hpp>

IDBFile idb(std::make_shared<std::ifstream>("database.i64", ios::binary));
ID0File id0(idb, idb.getsection(ID0File::INDEX));

uint64_t loadernode = id0.node("$ loader name");
std::cout << "Loader:" << '\t' << '\t'
	<< id0.getstr(loadernode, 'S', 0) << " - "
	<< id0.getstr(loadernode, 'S', 1) << std::endl;

uint64_t rootNode = id0.node("Root Node");

uint32_t version = id0.getuint(rootNode, 'A', -1);
uint32_t crc = id0.getuint(rootNode, 'A', -5);
time_t time = id0.getuint(rootNode, 'A', -2);
std::string str_version = id0.getstr(rootNode, 'S', 1303);

std::cout << "IDA Version:" << '\t' << version << "[" << str_version << "]" << std::endl
	<< "Time:" << '\t' << '\t' << time << std::endl
	<< "CRC:" << '\t' << '\t' << std::hex << crc << std::endl;
```

## Credits

The idb3 library was written by Willem Hengeveld <itsme@xs4all.nl>, and uses in [idbutil](https://github.com/nlitsme/idbutil) project. It is distributed under the MIT License.

## Third party

Using the [zlib library](https://zlib.net) for compressed databases.





