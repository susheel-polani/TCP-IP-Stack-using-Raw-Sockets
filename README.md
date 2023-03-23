# TCP-IP-Stack-using-Raw-Sockets

# Usage

python3 rawhttpget.py [url]

Example:
~: python3 rawhttpget.py http://david.choffnes.com/classes/cs5700f22/2MB.log
The above command downloads the 2MB.log file at the current directory.

# High Level Approach:

The rawhttpget program expects one command line argument, the URL of the GET request.

The first step of the program's logic is to fetch the source and destination IP addresses.

1. The local source IP address is fetched by appending ".local" to the socket.gethostname(). The resulting string is sent to gethostbyname() to fetch the actual source IP address.
2. The command line URL input is segmented into domain and path using the urllib.parse library. The extracted domain name is fed into gethostbyname() to obtain the destination IP address. The destination file is named appropriately based on the path extracted from the URL. If the path is empty or ends with '/', then the out file will be index.html

Next, the program picks a random number from (1024, 65535) and checks if the randomly generated port number is in use. If it is, then another random number is picked. If the chosen port number is not in use, that port number will be used for the TCP communication moving forward.

Next, the client program picks a random number from (0, 2^32 - 1) to be set as a sequence number. The acknowledgement number is set to zero for the time being until the server sets it during the handshake.

Next, the client program creates two RAW sockets, one for sending out packets and one for receiving packets.
Using the domain name and path extracted from the input URL, the HTTP GET message is constructed.

Soon after, the TCP Three-way handshake takes place by calling the threeWayHandshake(sendSocket, recvSocket) function.

1. Within the handshake function, the client sends out an SYN packet with seq_no set to the randomly generated value and ack_no set to zero.
2. The client waits for an SYN-ACK response from the server. seq_no and ack_no are extracted from the packet and checked if the write SYN message is ACKed and if the packet is destined to this program.
3. The client sends out an ACK packet in response to the SYN-ACK with seq_no as seq_no+1 and ack_no set to acknowledge the server's message. Additionally, the HTTP GET Request(as packet payload) is sent as another ACK packet to the server.
   Once the Server ACK is back, the file transfer has officially started.

Next, the getFileContent() function is called, which is responsible for receiving the HTTP data and sending appropriate ACKs back and writing the extracted HTTP response from the packet into the outfile buffer.

Once the file transfer is done, the file is saved locally.

# TCP/IP features implemented:

Every time a packet is sent out, an appropriate checksum is computed and packed into the TCP Header. This ensures the integrity of the packet sent.

Every time a packet is received, the TTL, source IP address and destination port are extracted.

1. TTL is checked for having a value greater than zero. This makes sure that the packet is not outdated.
2. The source IP address is checked against the initially computed server address. This ensures that the packet is from the communicating web server.
3. The destination port is checked against the port picked initially to start the communication. This ensures that the packet is destined for this specific program.

Once the client ensures that the packet indeed is destined to itself, the checksum of the received packet is extracted and replaced with zero. The checksum is recomputed on the client side and compared with the extracted one. If they are equal, the received packet has not been tampered with. Else, the packet is discarded.

The client program maintains the order of packets transmitted. If the webserver missed sending a certain expected sequence number, then the client catches it and keeps ACKing the previously received in-order packet from the server. Looking at this, the server retransmits the missed packet.

# Statistics

The client program rawhttpget was timed against three files:
1. http://david.choffnes.com/classes/cs5700f22/2MB.log
2. http://david.choffnes.com/classes/cs5700f22/10MB.log
3. http://david.choffnes.com/classes/cs5700f22/50MB.log

Time Performances for each log file:
2MB.log -> 6.38 seconds
10MB.log -> 18.03 seconds
50MB.log -> 1 minute 20 seconds

# Challenges Faced:

The initial rendition of my program was very resource heavy as I was saving the HTTP response in my program buffer instead of writing it into a file as soon as I received a response and cleared the buffer. This took a huge hit on the program's execution time for large HTTP files.

The Checksum function from silver moon only covered even length message byte arrays. I had to learn the functioning of the checksum function and handle the case for odd-length message arrays as well(just appended '\x00' at the end).

# Code Ownership:

I worked on this project solo.

# References

Most of my ideas and code are referenced from these two links:
1. https://www.binarytides.com/raw-socket-programming-in-python-linux/
2. https://gist.github.com/pklaus/856268/bfe1500ceb1f4d762f436db6df056912337e33cd
