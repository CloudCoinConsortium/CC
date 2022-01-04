# Requests to the RAIDA and Responses from the RAIDA have fixed headers that are standard. 



# ![#ff0](https://via.placeholder.com/15/ff0/000000?text=+) REQUESTS
All requests to the RAIDA have a Request Header and a Request Body. 

The Request Header is not encrypted.  The Header has a fixed length of 22 bytes. 
### 22 Byte Request Header Format: 
CL SP RI SH CM CM VE ID ID NO NO NO EC EC EN UD UD ID ID SN SN SN

### Request Header Byte by Byte:

index | Code | Name | Notes
---|---|---|---
0 | CL | Cloud ID  |  Future Use. 0 for now.
1 | SP | Split ID | Future Use if coins split. 0 for now.  
2 | RI | RAIDA ID  |  An integer between 0 and 24. Should match the RAIDA ID in the config file. 
3 |SH | SHARD ID  |  An integer between 0 and 31. Should match one of the the Shard IDs in the config file. 
4 |CM | Command byte 1 | 0x00 //will be zero for all numbers below 255
5 |CM | Command byte 2 | Will be the least significant number (This is the command number)
6 |VE | Command Version  | An integer that is zero but could be different as standards evolve.
7 |ID  |  Coin ID 1 | Allows many coins to be on the same RAIDA. Including CloudCoin, NFTs and Identification coins. 
8 |ID  |  Coin ID 2 | Least significant number
9  |NO | Nonce 1 / Key ID | The Nonce is used in AES CTR 128 Bit Encryption. There are 64 Bits in the Request Header. 
10 |NO | Nonce 2 / Key ID | The Key ID is used in the PUT KEY command as the identifier for the key in the key table. 
11 |NO | Nonce 3 / Key ID | 
12 |EC | Nonce 4 / Echo 1 | Client can send two bytes to the server and the server will echo these two bytes in the response.
13 |EC | Nonce 5 / Echo 2 | Another echo
14 |RE  | Reserved | This is for future use in case more info is needed to make UDP more reliable 
15 |UD  | Total UDP Packets |  This tells the RAIDA how many UPD packets are part of the request. The max is 64 (see How UDP works below) 
16 |EN  | Encryption  |  0x00 means no encryption. See encryption codes table.
17 |ID  |  Coin ID 1 | The coin ID of the coin used to encrypt (Most Significant Byte)
18 |ID  |  Coin ID 1 | The coin ID of the coin used to encrypt (Least Signficant Byte)
19 |SN  |  SN 1 / Nonce 6 /Key ID | The SN of coin whose AN was used as a shared secret for encryption.
20 |SN  |  SN 2 / Nonce 7 /Key ID|  
21 |SN  |  SN 3 / Nonce 8  /Key ID|
* Note: Many bytes do two jobs. The SN will be the serial number of the coin to be encrypted and part of the nonce. 

### Command Codes
Code| Command | Description
--- | --- 
 0 | POWN | Change ANs to PANs if authentic. 
 1 | Detect | Check if ANs are authentic. Do not change ANs.
 2 | Find | Check if AN or PAN from the last call is the AN in RAIDA
 3 | Fix | Fix A RAIDA who thinks the Coin is Counterfeit
 4 | Echo | See if the RAIDA is up and get response time. 
 5 | Validate Ticket | Check if Kerberos Ticket is authentic
 6 | Upload Encryption Key | Store a key meant for another RAIDA to use. 
 9 | Download Encryption Key | Get a Key that has been stored for the user.
 10 | Recover (Phase 2) | Recover lost passwords
 11 | Get Ticket | Get a Kerberos ticket that can be used to prove authenticy to other RAIDA
 15 | Version | The version of the RAIDA software
 19 | Report Lost (Phase II) | Allows user who lost coins to report this loss.
 30 | Free ID | Gives a coin that can be used as in ID to access software such as SkyVault. 
100	 | Deposit | Put coins in a SkyVault account. 
102	 | Deposit PANG | Put coins in a SkyVault account but generate PANs with a seed. 
104 |	Withdraw | Remove coins from a SkyVault
108 |	Transfer| Move coins to another SkyVault
110 |	Balance | See the total coins in a SkyVault account. 
114 |	Show Coins By Denomination| Show all the SNs of the coin in a SkyVault
115 |	Show Coins By Type | See just coins with a certain type. 
130 |	Show Statments| Show the transactions that have occured fro the SkyVault
131 |	Delete ALL Statments| Delete all transaction records
132 |	Show Payment| See if coins have arrived in the user's SkyVault
150 |	Sync Transfer Add| Add coins to a SkyVault that are missing. 
152 |	Sync Transfer Delete| Delete coins from a SkyVault that are orphans. 
160 |	Exchange Coin (Phase II) | Change Bitcoin Connect coin to CC
200 | NFT: Test Create  (Phase II)| N/A |
202 | NFT: Create  (Phase II)| 19 |
204 | NFT: Read (Phase II) | 20 |
208 | NFT: Delete (Phase II) | 35 |
210 | NFT: Count (Phase II) | 19 |
212 | NFT: Request Sync (Phase II) | N/A |
214 | NFT: Sync Response (Phase II) | N/A |
215 | Upgrade Coin | Turn an Old coin into a New CC. 


### Coin IDs

Code| Meaning 
--- | --- 
00 00 00 00 00 | Identification Coins
00 00 00 00 01 | CC
00 00 00 00 02 | NFT 


### Encryption Codes [See Encryption](https://github.com/worthingtonse/RAIDAX/blob/main/Encryption.md)
Code | Name | Description | Bytes after EN
---|---|-----|---
0 | No encryption | Clear Text | 5 ( All Zeros)
1 | 128 AES CTR | |  5 (2 Coin ID, 3 Serial Number)
2 | 128 AES CTR from Key Table | 5 ( 2 random number, 3 Key ID)



### Request Body Codes
Code | Bytes | Name | Notes
------|----|-------|-------
CH | 16 | Challenge | Used for mutual athentication. The Client sends a 4 byte number to the RAIDA. The RAIDA create a hash of this number and returns the first 4 bytes of this hash.
OW | 3  | Serial Number | 4 byte Integer. Serial Number of coin to be authenticated. 
AN | 16 | Authenticity Number | 16 bytes of random bytes.  
PA | 16 |  Proposed Authenticity Number  | The new AN if the AN is authentic (only used in POWN)
TK | 4 | Ticket  | Used only in Fix fracked RAIDA. This ticket comes from other RAIDA.  
EM | 256 | Email address (Used only in Recover) | A fixed 256 characters long string. The email address ends in a null code.  
SE | 8 | Seed | Used to create the ANs |  $seed2 = THIS_NODE_NUMBER + $sn + $se; $pan = md5($seed2);
KI | 16 | Key ID | Used to create a shared key |  A number between 11,0001 and 11,020,000
PG | 8 | Proposed Authenticity Number Generator | The PANG is used as a seed to calculate an AN so that the client does not have to download it. $seed2 = THIS_NODE_NUMBER + $sn + $pang; $pan = md5($seed2);
3E | 2 | End of Command | This tells the server that one command has ended and another may follow (will have only one command in phase 1 so this is just there for future use) This is "0011 1110" binary "3E" hex or	the character ">".

### Coin Type
Type Code | Name | Description
---|---|---
0 | Just Received | This coin is new and not been processed by the owner. Anyone can send a type 0
1 | Payment | This coin was sent for a specific reason. Check the memo for details
2 | RAIDA Payment | This coin was sent to pay a RAIDA
3 | Change | This coin can be used as change for people who send the coin to a merchant
4 | Public Change | This coin can be used by anyone to create change
5 | locked | This coin cannot be moved without the owner 100%
6 | processed | This coin has been changed by the owner 
200+ | Any Code over 200 are user defined

# ![#0FF](https://via.placeholder.com/15/0ff/000000?text=+)  RESPONSE
The response header is not encrypted.  The client must remember what order the coins were and the server must keep the order of the coins the same when responding.  

### 12 Byte Response Header Format: 
```
RI SH SS EX UD UD EC EC HS HS HS HS
```

### Response Header Codes
Code | Bytes  |  Name  |  Info
--- | ----|---- | ----
RI | 1 | RAIDA ID  |  Integer 0 through 24.
SH |  1 | SHARD ID  |  Integer 0 through 31. 
SS |   1 | Status  | See status codes.   
EX  |  1 |  Execution Time Micro Seconds |  Microseconds that it took to execute the query. Zero means less than one microsecond. 255 means more than 255 microseconds.  
UD |  2 | UDP Frame Count  |  This allows the client to know how to put the returned packets together in order. 
EC  |  2 |  Client Echo  | Returns exact info that Client supplied in the Request Header.  
HS  | 4 |  Hash of Challenge  |  The first four bytes of the MD5 hash of the requester's challenge bytes.  


### Response Body Codes
Code | Bytes  |  Name  |  Info
--- |---- | ---- | ----
MT | 4  |   Master Ticket  |  Used to help fix fracked raida. 
MS |  Varies  | Mixed Responses  |  If there is a mixed response, you will need one MS for every eight coins being powned. These are examined on the binary level. Sopose that nine coins were powned. There would need to be two bytes if a mixed reponse. Say that all coins failed except for the nith coin. The response would be 00000000 00000001. So the first byte shows the status of the first eight notes and the second byte shows the ninth with padding. Not that the first bit is on the right. 
ME TA | varies   |  Meta Data  |  This data is about the data that is being provided.
DA TA  | varies |   Data  |  If their is data associated with the coin, this code tells if the data is all the same for all the coins of if each coin has unique data. 


### Response Status Codes
Description                          | Decimal Code
-------------------------------------|---
Invalid cloud id			                  |   1
Raida is off line 			   			          |   2
Invalid UDP Frame count 	 				       |   15
Invalid packet length 	 				         |   16
Udp frame time out	 	 				           |   17
Wrong Raida 			 			                  |   18
Invalid Split Id 		 			              |   19
Shard not available   				           |   20
Validate - Ticket found				          |   21
Validate - Ticket not found			       |   22
Validate - Invalid Raida     		      |   23
Invalid Command 			  			             |   24
Coin Id not found 	  				            |   25
Coin limit exceded			 	  	           |   26
Invalid encryption code		 	  			     |   27
coin owner id not found		 	  			     |   28
Pown - AN and PAN are Same					      |   30
Len Of body Cant Div in coins		      |   32
End of request	Should have 0x3E 0x3E 				          |   33
Empty request body						             |   36
Validate - Ticket claimed earlier    |		 37
Validate - Ticket claimed				        |   38
Coin Number not found					           |   39
Free ID: SN already in use			        |   40
Free ID: Service Locked	             |   41
Free ID: SN was not greator than 26K	|   42
Failed to Authenticate				           |   64
Page not found						                 |   66
Break counter feit					              |   70
Break coins not found				            |   72
Break coins sum not match			         |   74
Join counter feit					               |   80
Join coins not found				             |   82
Join coins Sum not match			          |   84
Fix All Tickets Zero                 |   90
RAIDA Agent: Pages Returned          |   100
RAIDA Agent: No Changes              |   101
RAIDA Agent: Primary Up              |   102
RAIDA Agent: Primary Down            |   104
Mirror: Message Received             |   105
Mirror: Report Returned              |   106
Mirror: Requested File does not exist|   107
Show Statements: No Statements found | 120
Identify coin found					             |   192
identify coin not found				          |   193
Delete Statements: Statements Deleted | 195
Find all none						                  |   208
Find all AN							                   |   209
Fnd  all PA							                   |   210
Find mixed							                    |   211
Coin Converter: No Ticket						      |   212
Coin Converter: Serial Numbers Returned						                    |   213
All PASS							                      |   241
All FAIL							                      |   242
MIX									                         |   243
Show Payment: GUID Not Found					    |   244
Upgrade Coin: No Response from Legacy RAIDA       |   244
Coin Converter: No Ticket Found | 245
Coin Converter:Time out         | 246
SUCCESS								                      |   250
FAIL								                         |   251


## How The UDP Works
UDP is used instead of TCP. The UDP protocol can be made to be much more reliable by following this protocol.

1. The first packet that you send will contain the 22 byte header. All following packets will not cointain a header. 

2. Each packet will be no more than 1024 bytes long. This will keep the bytes from being seperated and arriving out of order. 

3. The Header Packet will contain the total amount of packets that are being sent in the 15th byte. 

4. There may not be any more than 64 packets sent in one request. 

5. It is up to the client to decide how many bytes each packet will have.

6. The last packet that is sent will contain the 0x3E 0x3E. These bytes may be used in the future or ECC purpose but are there now just as a place holder. 


