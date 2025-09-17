Client-Server Application in C
===================================

client.c
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ C
/*
	if this is Windows programming:
	"sock" should be of type SOCKET instead of int.
	Use closesocket instead of closesocket
	#include <winsock2.h> instead of all those unix headers
*/

#include <sys/types.h>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <netdb.h>
#include <memory.h>
#include <ifaddrs.h>
#include <net/if.h>
#include <errno.h>
#include <stdio.h> 
#include <stdlib.h>
// #include <iostream>  // C++

#include <iomanip>
#include <fcntl.h>

#include <thread>

#define PORT	"13123"
#define	MAX_DATA_SIZE	1024
#define	SA struct sockaddr



void *get_in_addr(struct sockaddr *sa)
{
	if(sa->sa_family == AF_INET) {
		return &(((struct sockaddr_in*)sa)->sin_addr);
	}
	
	return &(((struct sockaddr_in6*)sa)->sin6_addr);
}

void memzero(void *b, int len)
{
	volatile int x;
	char *a;
	a = (char *)b;
	for(x = 0; x < len; x++)
	{
		*(a+x) = 0;
	}
}

#include <atomic>

int sockfd;
bool running = true;
std::atomic<bool> connected(false);

static unsigned char mem_data[RECEIVE_MEM_SIZE];
std::atomic<size_t> memptr(0);

bool iterate_mem_ptr(size_t n)
{
	memptr += n;
	
	if(memptr >= data_size)
	{
		// memptr exceeded data size, reset
		memptr = 0;
		return false;
	}
	
	return true;
}

void send()
{
	std::vector<uint8_t> message;
	for(size_t i = 0; i < send_data_size; i++)
	{
		message.push_back(mem_send_data[i]);
	}
	
	printf("sending %d bytes..\n", message.size());
	
	int result = send(sockfd, &message[0], message.size() * sizeof(uint8_t), 0);
}

void receive()
{
	while(running)
	{
		if(connected == true)
		{
			int n = read(sockfd, &mem_data[memptr], 4 * sizeof(uint8_t));
			
			size_t beginAddr = memptr;
			
			if(n > 0)
			{
				uint16_t numberOfBytes = 0x00;
				numberOfBytes = mem_data[memptr + 0] << 8;
				numberOfBytes |= mem_data[memptr + 1];
				
				uint8_t msgType = mem_data[memptr + 2];
				
				iterate_mem_ptr(n);
				
				size_t wordCount = numberOfBytes / 4;
				for(size_t i = 1; i < wordCount; i++)
				{
					size_t n = read(sockfd, &mem_data[memptr], 4 * sizeof(uint8_t));
					if(n > 0)
					{
						iterate_mem_ptr(n);
					}
				}
			}
		}
	}
}


int main()
{
	if(!connected)
	{
		struct addrinfo hints, *servinfo, *p;
		int rv;
		char s[INET6_ADDRSTRLEN];
		
		memzero(&hints, sizeof hints);
		hints.ai_family = AF_UNSPEC;
		hints.ai_socktype = SOCK_STREAM;
		
		if((rv = getaddrinfo(address, PORT, &hints, &servinfo)) != 0)
		{
			// address not found
		}
		
		// loop through all the results and connect to the first we can
		for(p = servinfo; p != NULL; p = p->ai_next) {
			if ((sockfd = socket(p->ai_family, p->ai_socktype,
					p->ai_protocol)) == -1) {
				perror("client: socket");
				continue;
			}

			if (connect(sockfd, p->ai_addr, p->ai_addrlen) == -1) {
				perror("client: connect");
				close(sockfd);
				continue;
			}

			break;
		}

		if (p == NULL) {
			fprintf(stderr, "client: failed to connect\n");
			return 2;
		}
		else
		{
			inet_ntop(p->ai_family, get_in_addr((struct sockaddr *)p->ai_addr), s, sizeof s);
			fcntl(sockfd, F_SETFL, 0_NONBLOCK);
			
			// client connected to %s
			connected = true;
			
			freeaddrinfo(servinfo);
		}
	}
	else
	{
		// already connected
	}
	
	std::thread t(receive);
	// session loop
	{
	
		// send whenever requested
		// send();
	}
	
	// when all done
	running = false;
	if(close(sockfd) == 0)
	{
		printf("socket closed\n");
		connected = false;
	}
	
	t.join();
	return 0;
}

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~