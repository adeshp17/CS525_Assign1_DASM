
                                            Storage Manager
	                                        README

Storage Manager module is responsible for reading a block from file on the disk into memory and writting in-memory blocks to disk. It deals with fixed sized pages (blocks) of data and the size is specified by PAGE_SIZE. The size is configurable and the module needs to be built when this size is changed.

* Highlights
------------
* Seek optimization
* Well organized and designed code base.
* Access validations for internal functions so that they are not accessible from outside of storage manager like default APIs.
* Additional test routine testReadWrite() (in test_assign1_1.c) for thoroughly testing all the funtionalities. 

I. Contents
-----------
All source files (.c, .h)
README file
Makefile

II. File List
-------------
db_error.h		Utility functions and headers for return values and error codes
db_error.c		Implementation of utility functions
storage_mgr.h		Storage manager header
storage_mgr.c		Core Storage manager functionality implementation
test_assign1_1.c	Test file
my_test_assign1.c       Additional test file
test_helper.h           Headers and definitions for test files

III. Build and Run
------------------
A. Build
      1. For default build
	$ make clean
	$ make
      2. For additional test cases
        $ make clean
	$ make my_assign1

This will clean build the storage manager and create an executable named assign1(default)

B. Run
	$ ./assign1  (default)
	$ ./my_assign1 (additional test cases)


IV. Design and Implementation
-----------------------------

Features --
* Seek optimization
        To save on seek time, we first determine where to start seeking from (beginning or end of the file). This is determined based on whether the specified
		page position lies closer to the start of the file or to the end file.


A. Data Structures
	
* Page size 
	Page size is measured in bytes and is specified by PAGE_SIZE header. Page size is configurable. Current PAGE_SIZE is set to 4096 bytes.

* File Handle - SM_PageHandle
	Each file being accessed, needs to be created(if does not exist already) and opened first. Each opened file is then accessed using its
	file handle. File Handle contains important and handy metadata information about the file:
	- Name of the file being accessed
	- Total number of pages the file has
	- Current page which was accessed (read from/written into). It is counted from the begining of the file
	- Other management information as per need

* Page Handle - SM_PageHandle 
	A page handle is an pointer to an area in memory storing the data of a page. Read methods read data from a page on the disk into the
	area pointed to by this page handle. Similarly, write methods write data present in area pointed to by page handle to the disk.

* The initial bytes of the page file will contain the total number of pages the corresponding file has. This is the only metadata that we store in file. This metadata is then followed by the very first page of the file. We do not reserve specific number of initial bytes for storing this; instead it is made dynamic by seperating number of pages(metadata) and actual page data by '*' character. All the initial bytes upto '*' will indicate total number of pages.

* Page/Block index starts from 0.

B. Methods

* initStorageManager
	Initializes the storage manager. Additional tasks needed during the start of storage manager can be added in this routine.

* createPageFile  -Dhruvit Modi (dmodi2@hawk.iit.edu)
	Creates a new page file by the specified file name. If the file already exists then its contents will be discared and it will be treated
	as a new empty file. Size of the created file will be one page and total number of pages will be 1. The page will be filled with null
	bytes.
	Upon successful execution, RC_OK is returned else following error codes are returned -
	RC_FILE_NOT_FOUND
	RC_WRITE_FAILED

* openPageFile  -Dhruvit Modi (dmodi2@hawk.iit.edu)
	Opens the existing page file. If file does not exist then an error code is returned. File is opened in read/update mode such that it
	can be used by subsequent read/write operations. Before invoking any read/write operations of the storage manager, the corresponding
	file must be opened first using this method. Once the file is successfully opened, openPageFile fills the file handle structure with
	the file details. Total number of pages are read from initial bytes of the first page and then set in the structure. Current page will
	be set to the first page.
	Upon successful execution, RC_OK is returned else following error codes are returned -
	RC_FILE_NOT_FOUND

* closePageFile -Dhruvit Modi (dmodi2@hawk.iit.edu)
	Closes an open page file.
	Upon successful execution, RC_OK is returned else following error codes are returned -
	RC_FILE_HANDLE_NOT_INIT
	RC_FILE_CLOSE_FAILED

* destroyPageFile -Dhruvit Modi (dmodi2@hawk.iit.edu)
	Delete an existing open page file.
	Upon successful execution, RC_OK is returned else following error code is returned -
	RC_FILE_DESTROY_ERORR

* readBlock -Dhruvit Modi (dmodi2@hawk.iit.edu)
	Reads the pageNumth block from an already opened file into a buffer pointed to by page handle. The routine first checks if the specified page number is valid and exists. The file pointer is then seeked to the desired position. To save the seek time, we compute the distance 	of the desired page from the beginning of file and also from the end of file. Based on whichever is lesser, file pointer is either
	seeked from the beginning of the file or from the end.
	The curent page position is then set to the page which was read.
	Upon successful execution, RC_OK is returned else following error codes are returned -
	RC_FILE_HANDLE_NOT_INIT
	RC_READ_NON_EXISTING_PAGE
	RC_READ_ERROR

* readFirstBlock - Sahil Chalke (schalke@hawk.iit.edu)
	Reads the very first block from an already opened page file into a buffer pointed to by page handle.
	Upon successful execution, RC_OK is returned else following error codes are returned -
	RC_FILE_HANDLE_NOT_INIT
    RC_READ_ERROR

* readLastBlock -Anirudh Deshpande (adeshp17@hawk.iit.edu)
	Reads the last block from an already opened page file into a buffer pointed to by page handle.
        Upon successful execution, RC_OK is returned else following error codes are returned -
        RC_FILE_HANDLE_NOT_INIT
        RC_READ_ERROR

* readPreviousBlock -Anirudh Deshpande (adeshp17@hawk.iit.edu)
	Reads previous block of current block specified in file handle into a buffer pointed to by page handle.
	Upon successful execution, RC_OK is returned else following error codes are returned -
        RC_FILE_HANDLE_NOT_INIT
        RC_READ_ERROR

* readCurrentBlock  -Anirudh Deshpande (adeshp17@hawk.iit.edu)
	Reads the current block as specified in file handle into a buffer pointed to by page handle.
	Upon successful execution, RC_OK is returned else following error codes are returned -
        RC_FILE_HANDLE_NOT_INIT
        RC_READ_ERROR

* readNextBlock   -Anirudh Deshpande (adeshp17@hawk.iit.edu)
	Reads block which is next to the current block specified in file handle into a buffer pointed to by page handle.
	Upon successful execution, RC_OK is returned else following error codes are returned -
        RC_FILE_HANDLE_NOT_INIT
        RC_READ_ERROR

* writeBlock  -Sahil Chalke (schalke@hawk.iit.edu)
	Writes an in-memory block at the pageNumth position in the open page file. The page number specified must be valid and must exist.
	If the page does not exist then it's caller's responsibility to append a new page or call ensureCapacity to make sure that page does 
	exist before continuing with write operation.
	Upon successful execution, RC_OK is returned else following error codes are returned -
	RC_FILE_HANDLE_NOT_INIT
	RC_WRITE_FAILED
	
* writeCurrentBlock -Sahil Chalke (schalke@hawk.iit.edu)
	Writes an in-memory block at the page specified by current page position (present in file handle).
	Upon successful execution, RC_OK is returned else following error codes are returned -
        RC_FILE_HANDLE_NOT_INIT
        RC_WRITE_FAILED

* appendEmptyBlock  - Monika Priyadarshini (mpriyad1@hawk.iit.edu)
	Adds an empty block to the end of the page file. The block is filled with null bytes. It then updates the total number of pages in file 	handle and also in the metadata part of the file (initial few bytes of the page file).
	Upon successful execution, RC_OK is returned else following error codes are returned -
	RC_FILE_HANDLE_NOT_INIT
	RC_WRITE_FAILED

* ensureCapacity  - Monika Priyadarshini (mpriyad1@hawk.iit.edu)
	Ensures that page file has block count of at least numberOfPages. If the file does not already contain the specified number of pages
	then additional empty pages are added to ensure that file has the specified capacity.
	Upon successful execution, RC_OK is returned else following error codes are returned -
	RC_FILE_HANDLE_NOT_INIT
	
	
* getBlockPos  - Monika Priyadarshini (mpriyad1@hawk.iit.edu)
	Returns the current block number that the file handle points to. 
	Upon successful execution, RC_OK is returned else following error codes are returned -
	RC_FILE_HANDLE_NOT_INIT
	
* printFileHandle - Sahil Chalke (schalke@hawk.iit.edu)
  prints the file name, current position and total number of pages
  
 ----------------------------------------------------------------------------------------------------------------------------------------------------------------
 README : Monika Priyadarshini (mpriyad1@hawk.iit.edu)
 MAKEFILE: Sahil Chalke (schalke@hawk.iit.edu)
