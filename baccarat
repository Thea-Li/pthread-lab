#include <string.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <errno.h>
#include <ctype.h>
#include <pthread.h>
#include <sys/time.h>
#include <netdb.h>
#include <netinet/in.h>
#include <sys/file.h>

#define GO 0
#define STOP 1
#define DRAW 2


pthread_cond_t gamestate, playerhand, bankerhand;
pthread_mutex_t mutex;
int coutp, coutb, coutt;
int pc1,pc2,pc3,pr;
int bc1,bc2,bc3,br;
int BankerC = 0;
int CroupierP = 0;
int PlayerC = 0;
int CroupierB= 0;
int PlayerB = 0;
int gt = 0;


char *list[14]={"0","1","2","3","4","5","6","7","8","9","10","J","Q","K"};

void Croupier(){
	//first message sent part by croupier
	//let player and banker know it's time to start
	
	pthread_mutex_lock(&mutex);
	gt = GO;
	CroupierB = 1;
	CroupierP = 1;
	pthread_cond_broadcast(&gamestate);
	pthread_mutex_unlock(&mutex);
	
	//second part of croupier: wait for play and banker finished
	pthread_mutex_lock(&mutex);
	//printf("C waiting messsage from P\n");
		while (PlayerC == 0){
			pthread_cond_wait(&playerhand, &mutex);
		}
		//reset the status of player and banker
		//int localgt = gt;
		PlayerC = 0;
	pthread_mutex_unlock(&mutex);
	
	pthread_mutex_lock(&mutex);
	//printf("C waiting messsage from B\n");
		//check the croupier message status of player and banker 
	while (BankerC == 0){
		pthread_cond_wait(&bankerhand, &mutex);
	}
	BankerC = 0;
	pthread_mutex_unlock(&mutex);
	
	//printf("C Got  messsage from P and B\n");
	if ((pr == 9 || pr == 8 )&& (br != 8 && br != 9)) {
		printf("Player Wins!\n");
		coutp ++;
		gt = STOP;
		
	}
	else if ((br == 8 || br == 9 )&& (pr != 8 && pr != 9)){
		printf("Bank Wins!\n");
		coutb ++;
		gt = STOP;
	}
	//if there is a tie, set the message form croupier to player and banker to invalid. 
	else if ((br == 8 || br == 9 )&& (pr == 8 || pr == 9)){
		printf("Tie\n");
		CroupierP = 0;
		CroupierB = 0;
		gt = DRAW;
	}
	else{
		CroupierP = 0;
		CroupierB = 0;
		//printf("We need to draw third card player\n");
		gt = DRAW;
	}
	
	//pthread_mutex_unlock(&mutex);
	
	//As message sender , signal player , let player check the condition of new round or continue third card  
	//printf("Sending message to P and B\n");
	pthread_mutex_lock(&mutex);
	//printf("C GT %d",gt);
	CroupierP = 1;
	pthread_cond_signal(&gamestate);
	pthread_mutex_unlock(&mutex);
	
	//As a receiver, final result for round. 
	pthread_mutex_lock(&mutex);	
	//if both player and banker finished
	//printf("C wait for 3rd card result\n");
		
	while (PlayerC == 0){
		pthread_cond_wait(&playerhand, &mutex);
	}
	while (BankerC == 0) {
		pthread_cond_wait(&bankerhand, &mutex);
	}
	//printf("C GOT 3rd card result\n");
	BankerC = 0;
	PlayerC = 0;
	
	pthread_mutex_unlock(&mutex);

	if(br > pr){
		printf("Bank Wins!\n");
		coutb ++;
	}
	else if(br == pr){
		printf("Tie!\n");
		coutt ++;
	}
	else{
		printf("Player Wins!\n");
		coutp ++;
	}
	//pthread_mutex_unlock(&mutex);
}
	
void *Player(void *arg) {
	while (1) {
	int c1,c2,c3,r;
	//printf("Player ready to start\n");
	pthread_mutex_lock(&mutex);
	while (CroupierP == 0) {
		pthread_cond_wait(&gamestate, &mutex);
	}
	int localgt = gt;
	
	pthread_mutex_unlock(&mutex);
	
	if (localgt == GO) {
		
		c1 = (rand() % 13) + 1;
		c2 = (rand() % 13) + 1;

		printf("Player draws %s, %s \n",list[c1], list[c2]);
		//print JOK when the card result in any number larger than 10

		//if card num>=0, points are all 0
		if(c1 >= 10){
			c1 = 0;
		}
		if(c2 >= 10){
			c2 = 0;
		}
		//get result for player
		int r = c1 + c2;
		//if the cards sum larger than 10, get the rightmost number
		if (r >=10){
			r = r - 10;
		}
	
		//the message to croupier that player finished
		//printf("P is sending message back to Croupier\n");
		//As a message sender, player going to tell croupier that it finished
		
		pthread_mutex_lock(&mutex);
		//the message to croupier that player finished
		pc1 = c1;
		pc2 = c2;
		pc3 = 0;
		pr = r;
		PlayerC = 1;
		pthread_cond_signal(&playerhand);
		//PlayerB = 1;			
		//the message to croupier that player finished
		pthread_mutex_unlock(&mutex);
	}
		//printf("P is receiving message from Croupier\n");
		//As a message reveiver, it need to wait the signal from croupier
		
		pthread_mutex_lock(&mutex);
		while (CroupierP == 0) {
			pthread_cond_wait(&gamestate, &mutex);
		}
		localgt = gt;
		//printf("P GT %d",gt);
		CroupierP = 0;
		pthread_mutex_unlock(&mutex);
		
		if(localgt == DRAW) {
			r = pr;	
			if (r == 6 || r == 7 ){
				printf("Player Stands \n");
				c3 = 0;
			}
			
			else{
				c3 = (rand() % 13 + 1);
				printf("Player draws %s \n",list[pc3]);
				
			if(c3 >= 10){
				c3 = 0;
			}
			r += c3;
			//get the rightmost digit for result
			if (r >=10){
				r = r - 10;
			}	
			}
		
		//if(localgt == STOP){
		//	exit(0);
		//}
	
	pthread_mutex_lock(&mutex);
	//printf("P is sending message back to C fpr 3rd card\n");
	//printf("P is sending message to B for 3rd card\n");
	//the message to banker that player finished
	//message croupier that player finished
	pc1 = c1;
	pc2 = c2;
	pc3 = c3;
	pr = r;
	PlayerC = 1;
	PlayerB = 1;
	//localgt = gt;
	pthread_cond_broadcast(&playerhand);
	pthread_mutex_unlock(&mutex);
		}
	}
	return NULL;
}



void *Banker(void *arg) {
	while (1) {
	
	int c1,c2,c3,r;
	//printf("B is ready to start\n");
	
	pthread_mutex_lock(&mutex);
	while (CroupierB == 0) {
		pthread_cond_wait(&gamestate, &mutex);
	}
	int localgt = gt;
	//printf("B GT %d",gt);
	pthread_mutex_unlock(&mutex);

	pthread_mutex_lock(&mutex);
	if (localgt == GO) {
		//printf("B GOT signal from C\n");
		CroupierB = 0;
		c1 = (rand() % 13) + 1;
		c2 = (rand() % 13) + 1;		
		printf("Bank draws %s, %s \n",list[c1], list[c2]);
					
		if(c1 >= 10){
			c1 = 0;
		}
		if(c2 >= 10){
			c2 = 0;
		}
					//get result for player 
		int r = c1 + c2;
					//if the cards sum larger than 10, get the rightmost number
		if (r >=10){
			r = r - 10;
		}
	pthread_mutex_unlock(&mutex);
		
	//As the message sender, the banker sending message to croupier
	pthread_mutex_lock(&mutex);
	//printf("Banker is sending message back to C\n");
		//the message to croupier that banker finished
	bc1 = c1;
	bc2 = c2;
	bc3 = 0;
	br = r;
	BankerC = 1;
	pthread_cond_signal(&bankerhand);
	pthread_mutex_unlock(&mutex);
	}
	
	//As the message receiver, banker need to get a message from croupier that it need to wait for player
	//and then get a message from player that player finished 3rd card draw
	//printf("B try to get message from P\n");
	
	pthread_mutex_lock(&mutex);
	while(PlayerB == 0) {
		pthread_cond_wait(&playerhand, &mutex);
	}
	localgt = gt;
	//printf("B GT %d",gt);
	PlayerB = 0;
	//printf("B GOT message from P\n");
	pthread_mutex_unlock(&mutex);
	
	//set a flag for banker to decide if banker need to do 3rd card
	if(localgt == DRAW){
		
	int FLAGB = 0;
	r = br;
	if(r <= 2){
		FLAGB = 1;
	}
	else if (c3 != 8){
		if(r == 3){
			FLAGB = 1;
		}
		else if (r == 4 && (pc3 != 1)){
			FLAGB = 1;
		}
		else if (r == 5 && (pc3 != 1 || pc3 != 2 || pc3 != 3 )){
			FLAGB = 1;
		}
		else if ((r == 6 )&& ( pc3 == 6 || pc3 == 7)){
			FLAGB = 1;
		}
	}
				
	else{
		FLAGB = 0;
		printf("Banker stands\n");
	}
			//check the flag of banker and do 3rd card
	if(FLAGB == 1){
		
		c3 = (rand() % 13) + 1;
		printf("Bank draws %s \n",list[bc3]);	
				
		if(c3 >= 10){
			c3 = 0;
		}		
		r += c3;
		if (r >=10){
			r = r - 10;
		}
	}
	
	//else if(localgt == STOP){
	//		exit(0);
	//	}

	//As a message sender, it need to tell banker that it finished. 
	//if(PlayerC == 0){
	pthread_mutex_lock(&mutex);
	//the message to croupier that banker finished
	bc1 = c1;
	bc2 = c2;
	bc3 = c3;
	br = r;
	localgt = gt;
	BankerC = 1;
	pthread_cond_signal(&bankerhand);
	pthread_mutex_unlock(&mutex);
	}
	//BankerC = 0;
	}
	return NULL;
}


int main(int argc, char **argv){
	//input how many round they want
	//the round should be only in croupier but not in banker /player 
	int round = atoi(argv[1]);

	printf("Beginning %d Round... \n",round);	
	pthread_t threadp,threadb;		
	
	if (pthread_create(&threadp, NULL,Player, NULL) != 0){
		perror(0);
	}
	
	if (pthread_create(&threadb, NULL,Banker, NULL) != 0){
			perror(0);
	}	
		
	for(int i = 0; i < round; i ++){
		printf("Round %d:  \n",i+1 );
		//for croupier to signal how many times player and banker can go
		Croupier();	
		printf("----------------------- \n");
	}
			
	printf("Result: \n");
	printf("Player: %d \n", coutp);
	printf("Bank  : %d \n", coutb);
	printf("Ties  : %d \n", coutt);
			
	if((coutp > coutt) && (coutp > coutb)){
		printf("Player Wins!\n");
	}
	else if((coutb > coutt) && (coutb > coutp)){ 
		printf("Bank Wins!\n");
	}
	else if((coutt > coutp) && (coutt > coutb)){
		if(coutb > coutp){
			printf("Bank Wins!\n");
		}
		else if (coutp > coutb){
			printf("Player Wins!\n");
		}
	}	
	else {
		printf("Tie!!!!\n");
	}		
		
//	pthread_mutex_destroy(&mutex);
//	pthread_cond_destroy(&gamestate);
//	pthread_cond_destroy(&playerhand);
//	pthread_cond_destroy(&bankerhand);

	pthread_join(threadp, NULL);
	pthread_join(threadb, NULL);
			
//	pthread_exit(NULL);
//	pthread_exit(NULL);
		
	return 0;

}
			
					

