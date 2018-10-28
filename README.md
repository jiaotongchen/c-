#include<stdio.h>
#include<stdlib.h>
#include<pthread.h>
#include<unistd.h>

int count1=0,count2=0,count3=0;
int data=1000;
typedef struct node
{
    int data;
    struct node* next;
}node_t,*node_p,**node_pp;

node_p head=NULL;

pthread_cond_t cond=PTHREAD_COND_INITIALIZER;
pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;


node_p AllocNode(int d,node_p node)
{
    node_p n=(node_p)malloc(sizeof(node_t));
    if(!n)
    {
        perror("malloc");
        exit(1);
    }

    n->data=d;
    n->next=node;
    return n;

}

void InitList(node_pp _h)
{
    *_h=AllocNode(0,NULL);
}

int IsEmpty(node_p list)
{
    if( NULL == list->next)
    {
        return 1;
    }
    else
    {
        return 0;
    }
}

void FreeNode(node_p n)
{
    if(n!=NULL)
    {
        free(n);
        n=NULL;
    }
}

void PushFront(node_p n,int d)
{
    node_p tmp=AllocNode(d,NULL);
    tmp->next=n->next;
    n->next=tmp;
}

void PopFront(node_p n,int* out)
{
    if(!IsEmpty(n))
    {
        node_p tmp=n->next;
        n->next=tmp->next;
        *out=tmp->data;
        FreeNode(tmp);
    }
}

void ShowList(node_p n)
{
    node_p begin=n->next;
    while(begin)
    {
        printf("%d ",begin->data);
        begin=begin->next;
    }
    printf("\n");
}

void Destory(node_p n)
{
    int data;
    while(!IsEmpty(n))
    {
        PopFront(n,&data);
    }
    FreeNode(n);
}

int Tuipiao()
{
   int sc=rand()%100;
   
   if(((sc%10)==3)|((sc%10)==2)|((sc%10)==1)){
         return rand()%10+1;
   }else{
	return 0;
   }
}

int isb()
{
  int sc=rand()%10;
  if((sc==3)|(sc==2)|(sc==1)|(sc==8)|(sc==9)|(sc==10)){
         return 1;
   }else{
		return 0;   
   }
}

void* Tianjin(void* arg)
{
  while(1)
   {		
		//锁
      pthread_mutex_lock(&lock);
    
	   if(isb())
        {
		   //开锁
        pthread_mutex_unlock(&lock);
        pthread_cond_signal(&cond);
		  continue;
		}else{
			int i=Tuipiao();
		    data=i+data;
		int da = rand()%10;
		if(da>data){
		   printf("天津站买了:%d,退票：%d,还剩：0 \n",data,i);	  
		   count1=count1-i+data;
		    data=0;          
		}else{
		   count1=count1+da-i;
           data=data-da;
		   printf("天津站买了:%d,退票：%d,还剩：%d\n",da,i,data);	
		   
		}
        PushFront(head,data);
        pthread_mutex_unlock(&lock);//开锁
        pthread_cond_signal(&cond);   		 
    }
   if(data<=0){	   
	      break;
	   }      
  }
}

void* Beijing(void* arg)
{
    while(1)
    { 
     pthread_mutex_lock(&lock);
      if(isb())
        {
        pthread_mutex_unlock(&lock);//开锁
        pthread_cond_signal(&cond);
		  continue;
		}else{
		  int ii=Tuipiao();
		  data+=ii;
          int da1 = rand()%10;
          if(da1>data){
		  printf("北京站买了:%d,退票：%d，还剩：0\n",data,ii);
		   data=0;
		   count2=count2-2*ii+data;   
		}else{
          data=data-da1;
      
		 printf("北京站买了:%d,退票：%d,还剩：%d\n",da1,ii,data);
		 count2=count2+da1-ii;
		}
         
        pthread_mutex_unlock(&lock);
        pthread_cond_signal(&cond);
		 
	   }
     if(data<=0){	   
	      break;
	   }
    }
}


void* Shanghai(void* arg)
{
    while(1)
    {	   
		//锁
      pthread_mutex_lock(&lock);
    
	   if(isb())
        {
		   //开锁
        pthread_mutex_unlock(&lock);
        pthread_cond_signal(&cond);
		  continue;
		}else{
			int i2=Tuipiao();
		    data=i2+data;
		int da2 = rand()%10;
		if(da2>data){
		   printf("上海站买了:%d,退票：%d,还剩：0 \n",data,i2);	  
		   count3=count3-2*i2+data;
		    data=0;
       
		}else{
		   count3=count3+da2-i2;
           data=data-da2;     
		   printf("上海站买了:%d,退票：%d,还剩：%d\n",da2,i2,data);	
		
		}	 
        PushFront(head,data);
        pthread_mutex_unlock(&lock);//开锁
        pthread_cond_signal(&cond); 		
    }   

	if(data<=0){	   
	      break;
	   }    
  }
}

int tick=100;//写数据
pthread_rwlock_t rwlock;//定义读写锁

void *Counter(void *arg)//读者
{   

    while(tick>=0)
    {  
        if(pthread_rwlock_tryrdlock(&rwlock)!=0)//读者加锁失败，则说明写者正在写
        {
            printf("已经数数，数了%d张票\n",tick);
            

        }
        else
        {
        	
        printf("Counte正在数数\n"); 
        pthread_rwlock_unlock(&rwlock);//解锁

        }
        if(tick<=0){
		break;
		}
    }

}
void *Pep(void *arg)//人
{
     sleep(1);  //柜台先运行
     
     while(tick>=0)
     {  
         if(pthread_rwlock_trywrlock(&rwlock)!=0)//写者加锁失败，则说明读者正在读
        {
          printf("count可以来了...\n");
           
        }
        else
        {
         tick--;
         printf("Pep数数.%d\n",tick);
         pthread_rwlock_unlock(&rwlock);

        }
         if(tick<=0){
		break;
		}
     }

}


int main()
{
   int countt=0;
  //while(countt<2){
//	
//	int sw;
//	printf("输入1 or 2\n");
//	int a;
//	scanf("%d",a);
//	sw=a;
//
//   // if (sw==1){
//     printf("输入");
//  	countt++;
//  	
    pthread_mutex_init(&lock,NULL);

    InitList(&head);
    pthread_t beijing,shanghai,tianjin;
    pthread_create(&beijing,NULL,Beijing,NULL);
	pthread_create(&shanghai,NULL,Shanghai,NULL);
    pthread_create(&tianjin,NULL,Tianjin,NULL);

    pthread_join(beijing,NULL);
	pthread_join(shanghai,NULL);
    pthread_join(tianjin,NULL);
  
    Destory(head);
    pthread_mutex_destroy(&rwlock);//销毁锁
    
   printf("天津站退了:%d\t,北京站买了：%d\t,上海站买了：%d\n",count1,count2,count3);
   
  printf("===================================================================\n");

  


   pthread_rwlock_init(&rwlock,NULL);//初始化读写锁
   
   pthread_t counter,pep;
   pthread_create(&counter,NULL,Counter,NULL);
   pthread_create(&pep,NULL,Pep,NULL);

   pthread_join(counter,NULL);
   pthread_join(pep,NULL);

   pthread_rwlock_destroy(&rwlock);//销毁读写锁


	return 0;
	
}




  



