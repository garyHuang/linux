###1��redis���
     redis��һ��key-value�洢ϵͳ����Memcached���ƣ���֧�ִ洢��value������Ը��࣬����string(�ַ���)��list(����)��set(����)��zset(sorted set --���򼯺�)��hash����ϣ���ͣ�����Щ�������Ͷ�֧��push/pop��add/remove��ȡ���������Ͳ�����ḻ�Ĳ�����������Щ��������ԭ���Եġ��ڴ˻����ϣ�redis֧�ָ��ֲ�ͬ��ʽ��������memcachedһ����Ϊ�˱�֤Ч�ʣ����ݶ��ǻ������ڴ��С��������redis�������ԵİѸ��µ�����д����̻��߰��޸Ĳ���д��׷�ӵļ�¼�ļ��������ڴ˻�����ʵ����master-slave(����)ͬ����

###2����װ
####2.1�������������ѹĿ¼�������ѹĿ¼��ʹ��make����
```
make
#�����Ƿ�make�ɹ�
make test 
#��װ�� /usr/local/redis Ŀ¼��
make PREFIX=/usr/local/redis install
```
####3������˵��
> * daemonize �����Ҫ�ں�̨���н�ֵ����Ϊyes
> * pidfile ���ö��pid��ַ������/var/run/redus.pid
> * dir ���������ļ������Ŀ¼
> * bind ��ip�����ú�ֻ������֮��ip�ķ���
> * port �����˿ڣ�Ĭ��Ϊ6379
> * timeout ���ÿͻ������ӳ�ʱʱ�䣬��λΪ��
> * loglevel ��Ϊ4���� debug �� verbose �� notice �� waring
> * logfile ��־�ļ���ַ
> * databases �������ݿ����Ĭ��ʹ�����ݿ�Ϊ16
> * save ����redis�������ݿ⾵���Ƶ��
> * dbfilename ���ݿ��ļ�����
> * requirepass ����redis����
> * 

####4��api˵��
```
set name lijie //����һ��nameΪlijie�ļ�ֵ��
setnx name lijie //����һ����ֵ�ԣ����name�����ǲ����ڵ�
setex name 10 red //����һ����ֵ�ԣ���Ч����10����
keys map* //�鿴����map��ͷ�ļ���֧������
EXISTS name //�ж�name���Ƿ���ڣ����ڷ���1�������ڷ���0
del name  //ɾ����ô�����ɹ�����1��ʧ�ܷ���0
expire name 10 //����key�Ĺ���ʱ��,���ù���ʱ��Ϊ10��
PERSIST name //ȡ������ʱ��
ttl name  //��ȡ����ʱ�䣬����-2�Ѿ����ڻ��߲����ڣ�����-1��û�����ù���ʱ��
rename name newName //������
type name //���ؼ�����������
ping //����redis�ͻ����Ƿ�����������redis��������
CONFIG GET timeout //����redis���õ�ֵ
FLUSHDB //��յ�ǰ���ݿ�����м�
flushall //����������ݿ��е����м�
select 1 //�л����ݿ⣬����1�����ݿ�����
```
####5����鼯Ⱥ�ӳ�
./redis-cli -h 192.168.1.21 --latency

####6��redis�����˺�����
�޸������ļ����ҵ� requirepass foobared������������һ��(ע��������������ø���һ�㣬redisһ���ֿ��Խ���150K�ε����󣬼�����������ƽ�)
```
requirepass 123456
```
������ɺ�����
������ɺ󣬿ͻ������ӣ�
./redis-cli -h 192.168.1.21 -a 123456
����: ./redis-cli -h 192.168.1.21 ������ auth 123456
####7��redis������
MULTI //�������񣬽���������ִ�������������,���¡�
set a 10026
incr a
exec //ֱ���ύ����
DISCARD //�ع�����
####8��redis�ֹ���
 WATCH a //���ĳ��key
 muti    // ��������
 set a  b //���ò�һ����ֵ
 exec    //�ύ���񣬵�ִ����Ϻ󣬻��Զ��ͷ�watch������ٴμ�����ֵ������Ҫ���¼��,����ύ�����ʱ����a��ֵ�Ѿ����޸��ˣ��򲻻��ٴ��޸�a��ֵ

####9�� appendonly
 ������appendonly��ֵΪyes��ʱ��ϵͳ���Զ���ÿ��д��������뵽appendonly.aof���ļ��С�
 д������ļ��Ĺ����ǣ�
 appendfsync everysec //ÿ����д��һ��
 appendfsync always //ʵʱд��
 appendfsync no  //�Ӳ�д��

####11��redis�㲥�Ͷ���
SUBSCRIBE tv1 //����ĳ��key�������Ƕ��
PUBLISH tv1 hello //��tv1����hello��Ϣ���������ж���tv1�Ŀͻ��˶��ܽ��յ������Ϣ��