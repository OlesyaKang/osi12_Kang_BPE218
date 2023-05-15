# osi12-Kang-BPE218
##  12-ая домашняя работа Кан Олеся
### Код программы-клиента на языке си

## В этой домашней рабте требовалось разработать клиент-серверное приложение, использующее UDP и реализующее широковещательную рассылку множества сообщений с сервера. Сообщения с сервера в цикле набираются в консоли и передаются клиентам, запущенным на разных компьютерах. Решение достаточно реализовать на локальной сети. Завершение работы сервера и клиентов в данном случае не оговаривается.

### Код программы-клиента
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <unistd.h>

#define SERVER_ADDR "127.0.0.1"
#define PORT 8888 
#define MAX_MSG_LEN 1024 

int main() {
    int s; 
    char buf[MAX_MSG_LEN];
    struct saddr_in serv_addr, cli_addr; 
    slen_t serv_len = sizeof(serv_addr); 
    int broadcast = 1; 

    if ((s = socket(AF_INET, SOCK_DGRAM, IPPROTO_UDP)) < 0) {
        perror("socket error");
        exit(EXIT_FAILURE);
    }

    if (setsockopt(s, SOL_SOCKET, SO_BROADCAST, &broadcast, sizeof(broadcast)) < 0) {
        perror("setsockopt error");
        exit(EXIT_FAILURE);
    }

    memset(&serv_addr, 0, sizeof(serv_addr));
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_addr.s_addr = INADDR_ANY;
    serv_addr.sin_port = htons(PORT);

    if (bind(s, (const struct saddr *)&serv_addr, sizeof(serv_addr)) < 0) {
        perror("bind error");
        exit(EXIT_FAILURE);
    }

    printf("Client run.\n");

    while (1) {
        if (recvfrom(s, buf, MAX_MSG_LEN, 0, (struct saddr *)&serv_addr, &serv_len) < 0) {
            perror("recvfrom error");
        } else {
            printf("Received message: %s", buf);
        }
    }

    close(s);
    exit(EXIT_SUCCESS);
}
```

### Код программы-сервера
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <unistd.h>

#define SERVER_ADDR "127.0.0.1" 
#define PORT 8888 
#define MAX_MSG_LEN 1024

int main() {
    int s; 
    char buf[MAX_MSG_LEN]; 
    struct saddr_in serv_addr, cli_addr; 
    slen_t cli_len = sizeof(cli_addr); 
    int broadcast = 1; 

    if ((s = socket(AF_INET, SOCK_DGRAM, IPPROTO_UDP)) < 0) {
        perror("socket error");
        exit(EXIT_FAILURE);
    }

    if (setsockopt(s, SOL_SOCKET, SO_BROADCAST, &broadcast, sizeof(broadcast)) < 0) {
        perror("setsockopt error");
        exit(EXIT_FAILURE);
    }

    memset(&serv_addr, 0, sizeof(serv_addr));
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_addr.s_addr = inet_addr(SERVER_ADDR);
    serv_addr.sin_port = htons(PORT);

    if (bind(s, (const struct saddr *)&serv_addr, sizeof(serv_addr)) < 0) {
        perror("bind error");
        exit(EXIT_FAILURE);
    }

    printf("Server run.\n");

    while (1) {
        fgets(buf, MAX_MSG_LEN, stdin);

        if (sendto(s, buf, strlen(buf), 0, (const struct saddr *)&serv_addr, sizeof(serv_addr)) < 0) {
            perror("sendto error");
        }
    }

    close(s);
    exit(EXIT_SUCCESS);
}
```
