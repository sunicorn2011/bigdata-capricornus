[toc]

# HDFS����
## HDFS����
- ���壺hdfs���ļ��洢ϵͳ��ͨ��Ŀ¼������λ������Ƿֲ�ʽ
- ���ó�����
    - �ʺ�һ��д�룬��ζ������Ҳ�֧���ļ��޸ġ�
    - hdfsֻ�ܹ��ǵ��߳�д��
## HDFS��֯�ܹ�
- �ܹ�ͼ
    - ![image](1CCE1DF858CD47078BCCBB3FBE7B291A)
## �ܹ�ͼ��ɫ
- NameNode��NN��
    - ϵͳMaster��ɫ
    - ����
        - ����hdfs�������ռ�
        - ���ø�������
        - �������ݿ⣨Blockӳ����Ϣ��
        - ����ͻ�������
- DataNode��DN��
    - ���ݴ洢��ʵ��ִ����
    - ����
        - �洢ʵ�ʵ����ݿ�
        - ִ�����ݵĶ�д����
- Client��
    - ����
        - �ļ���Ƭ
        - ��NN��������ȡ�ļ��洢λ��
        - ��DN�������������ݵĶ�д����
        - �ṩһЩ���������ʸ���hdfs����ʽ������ɾ�Ĳ��
- Secondary NameNode
    - ��NN���ȱ���ɫ����NN�ҵ����������������ṩ����
    - ���ã�
        - �ֵ�NN�Ĺ��������綨�ڵĺϲ�fsimage��edit�༭��־���������µ�fsimage���͸�NN
        - ��������£����Ը����ظ�NameNode
        - 
## HDFS�ļ����С
- hdfs����ֿ飨block����С��128M��Ĭ��ͨ��dfs.blocksize������
- ������ʵ��
```
    max(minSize, min(maxSize, default_size(128M)))
```
- HDFS�п��С����Ҫ����ȡ���ڴ��̴���Ч��
    - ԭ��
        - 1���������Ѱַʱ��Ϊ10ms��������Ŀ��飨block����ʱ��10ms
        - 2��ԭ����Ѱַʱ��Ϊ����ʱ��� `1%`Ϊ���״̬������ʱ��Ϊ ` 10ms / 1% = 1000ms`
        - 3����Ŀǰ���̵��ձ鴫��Ч���� 100MB/s
        - 4������Block��С = 1s * 100MB/s = 100MB
## HDFS��ȱ��
- �ŵ㣺
    - ���ݴ���
        - �����ж������
    - �ʺϴ��������
        - ���ݹ�ģ���ļ���ģ 
    - �ɹ��������ۻ������棬ͨ���ั�����ƣ���߿ɿ���
- `ȱ��`��
    - ���ʺϵ��ӳٷ���
    - �޷���Ч�ĶԴ���С�ļ����д洢
        - �洢����С�ļ��Ļ�����ռ��NameNode�������ڴ����洢�ļ�Ŀ¼�Ϳ���Ϣ����������ȡ
        - С�ļ��洢��Ѱַʱ��ᳬ����ȡʱ�䣬��Υ����HDFS�����Ŀ��(`Ѱַʱ���Ƕ�ȡʱ��� 1% ����`)
        - 
    - ��֧�ֲ���д����ļ�������޸�
        - һ���ļ�ֻ����һ��д����������߳�д��
        - ��֧������append����֧���ļ�����޸�
# HDFS��������д��/��ȡ��
## д��
- ������ͼ
    - ![image](96C7BC1427FA40C6A2456E4D1642BACE) 
- д������:
    - �ͻ���ͨ��Distributed FileSystemģ����NameNode�����ϴ��ļ���NameNode���Ŀ���ļ��Ƿ��Ѵ��ڣ���Ŀ¼�Ƿ���ڡ�
    - NameNode�����Ƿ�����ϴ���
    - �ͻ��������һ�� Block�ϴ����ļ���DataNode�������ϡ�
    - NameNode����3��DataNode�ڵ㣬�ֱ�Ϊdn1��dn2��dn3��
    - �ͻ���ͨ��FSDataOutputStreamģ������dn1�ϴ����ݣ�dn1�յ�������������dn2��Ȼ��dn2����dn3�������ͨ�Źܵ��������
    - dn1��dn2��dn3��Ӧ��ͻ��ˡ�
    - �ͻ��˿�ʼ��dn1�ϴ���һ��Block���ȴӴ��̶�ȡ���ݷŵ�һ�������ڴ滺�棩����PacketΪ��λ��dn1�յ�һ��Packet�ͻᴫ��dn2��dn2����dn3��dn1ÿ��һ��packet�����һ��Ӧ����еȴ�Ӧ��
        - `�ص�`���˴�������������Ϣ���У�packet��Ϣ���к�ACK��Ϣ����
    - ��һ��Block�������֮�󣬿ͻ����ٴ�����NameNode�ϴ��ڶ���Block�ķ����������ظ�ִ��3-7������
        - `�ο�Դ��`��
        ```
        org.apache.hadoop.hdfs.DFSOutputStream
        
        ```
- ��������-�ڵ�������
    - ��HDFSд���ݵĹ����У�NameNode��ѡ�������ϴ�������������DataNode�������ݡ�
    - ����ѡ���ԭ��
        - ԭ����ѭ`���ܸ�֪�������洢�ڵ�ѡ�� `
        - `�ڵ���룺�����ڵ㵽������Ĺ�ͬ���ȵľ����ܺ͡�` 
    - `�����ڵ�ѡ��ԭ��`
        - ͼʾ![image](315508D7B39C48EA919763B2E4AE7FA5)
        - ѡ�����£�
            - ��һ��������Client�����Ľڵ��ϣ�����ͻ����ڼ�Ⱥ�⣬�����һ��
            - �ڶ�������������һ�����������
            - �����������ڵڶ����������ڵ�����ڵ�
## ��ȡ
- ������ͼ
    - ![image](CC0AA535B0EE442CA941723A076A8107)
- ��ȡ����:
    - �ͻ���ͨ��DistributedFileSystem��NameNode���������ļ���NameNodeͨ����ѯԪ���ݣ��ҵ��ļ������ڵ�DataNode��ַ��
    - ��ѡһ̨DataNode���ͽ�ԭ��Ȼ��������������������ȡ���ݡ�
    - DataNode��ʼ�������ݸ��ͻ��ˣ��Ӵ��������ȡ��������������PacketΪ��λ����У�飩��
    - �ͻ�����PacketΪ��λ���գ����ڱ��ػ��棬Ȼ��д��Ŀ���ļ���
    - 
# NameNode��SecondaryNameNode��ϵ
## NN��2NN��������
    - ԭ��
        - ����һ���µĽڵ�SecondaryNamenode��ר������FsImage��Edits�ĺϲ��� 
## NameNode�Ĺ���ԭ��
- �ṹͼ 
    ![image](6964A8A916804581B5BCED498D85141B)
- ���̣�
        - ��һ�׶Σ�NameNode����
            - ��һ������NameNode��ʽ���󣬴���Fsimage��Edits�ļ���������ǵ�һ��������ֱ�Ӽ��ر༭��־�;����ļ����ڴ档
            - �ͻ��˶�Ԫ���ݽ�����ɾ�ĵ�����
            - NameNode��¼������־�����¹�����־��
            - NameNode���ڴ��ж�Ԫ���ݽ�����ɾ�ġ�
            - 
        - �ڶ��׶Σ�Secondary NameNode����
            - Secondary NameNodeѯ��NameNode�Ƿ���ҪCheckPoint��ֱ�Ӵ���NameNode�Ƿ�������
            - �����Ҫ����Secondary NameNode����ִ��CheckPoint��
            - NameNode��������д��Edits��־���������µ�Edit��־
            - ������ǰ�ı༭��־�;����ļ�������Secondary NameNode��
            - Secondary NameNode���ر༭��־�;����ļ����ڴ棬���ϲ���
            - �����µľ����ļ�fsimage.chkpoint��
            - ����fsimage.chkpoint��NameNode��
            - NameNode��fsimage.chkpoint����������fsimage��
## CheckPointʱ������
- ͨ������£�SecondaryNameNodeÿ��һСʱִ��һ��
        - �ļ���`hdfs-default.xml` 
        ```
        <property>
          <name>dfs.namenode.checkpoint.period</name>
          <value>3600s</value>
        </property>
        ```
    - һ���Ӽ��һ�β��������������������ﵽ1����ʱ��SecondaryNameNodeִ��һ��
        - �ļ�: 
        ```
        <property>
          <name>dfs.namenode.checkpoint.txns</name>
          <value>1000000</value>
        <description>������������</description>
        </property>
        
        <property>
          <name>dfs.namenode.checkpoint.check.period</name>
          <value>60s</value>
        <description> 1���Ӽ��һ�β�������</description>
        </property >
        ```
-
## NameNode���ϴ�����չ��
- ����
    - ����һ����SecondaryNameNode�����ݿ�����NameNode�洢���ݵ�Ŀ¼
        - kill -9 NameNode����
        - ɾ��NameNode�洢�����ݣ�/opt/module/hadoop-3.1.3/data/tmp/dfs/name��
        - ����SecondaryNameNode�����ݵ�ԭNameNode�洢����Ŀ¼
        - ��������NameNode
    - ��������
        - ʹ��-importCheckpointѡ������NameNode�ػ����̣��Ӷ���SecondaryNameNode�����ݿ�����NameNodeĿ¼�С�
    - 
# ��Ⱥ��ȫģʽ
- �ṹͼ
![image](DB8F4BEE378C4639A1D257EBC8ACF5F0)
- �����﷨
```
��1��bin/hdfs dfsadmin -safemode get	�������������鿴��ȫģʽ״̬��
��2��bin/hdfs dfsadmin -safemode enter  ���������������밲ȫģʽ״̬��
��3��bin/hdfs dfsadmin -safemode leave	�������������뿪��ȫģʽ״̬��
��4��bin/hdfs dfsadmin -safemode wait	�������������ȴ���ȫģʽ״̬��
```
# DataNode
## DataNode��������
- ����ͼ
- ![image](A9A1AE6EFE334E99B4D3AA94DA82FF81)
- DataNode�����������̣�
    - һ�����ݿ���DataNode�����ļ���ʽ�洢�ڴ����ϣ����������ļ���һ�������ݱ���һ����Ԫ���ݰ������ݿ�ĳ��ȣ������ݵ�У��ͣ��Լ�ʱ�����
    - DataNode��������NameNodeע�ᣬͨ���������ԣ�1Сʱ������NameNode�ϱ����еĿ���Ϣ��
    - ������ÿ3��һ�Σ��������ؽ������NameNode����DataNode�������縴�ƿ����ݵ���һ̨��������ɾ��ĳ�����ݿ顣�������10����30��û���յ�ĳ��DataNode������������Ϊ�ýڵ㲻���á�
    - ��Ⱥ�����п��԰�ȫ������˳�һЩ������
- DataNode����������У�鷽��
    - 1����DataNode��ȡBlock��ʱ���������CheckSum��
    - 2�����������CheckSum����Block����ʱֵ��һ����˵��Block�Ѿ��𻵡�
    - 3��Ȼ��Clientȥ��ȡ����DataNode�ϵ�Block��
    - 4��������У���㷨 crc��32����md5��128����sha1��160��
    - 5��DataNode�����ļ�������������֤CheckSum��
- ����ʱ�޲�������
    - ���̣�
    - ![image](6E21C77B02AB408B81E5A8CD6B7E6B83)
    - ���ó�ʱʱ�䣺
        - ʱ�����㣺TimeOUT = 2*recheck-interval + 10*interval
        - �ļ���`hdfs-site.xml`
        ```
        <property>
            <name>dfs.namenode.heartbeat.recheck-interval</name>
            <value>300000</value ---- ����
        </property>
        <property>
            <name>dfs.heartbeat.interval</name>
            <value>3</value>        ---- ��
        </property>
        ```
- ���������ݽڵ�
    - ���۲��裺
        - 1����¡һ̨DataNode�ڵ���� 
        - 2��ֱ������DataNode�����ɹ�������Ⱥ
            - 1��������
            ```
                hdfs --daemon start datanode
                yarn --daemon start nodemanager
            ```
        - 3����hadoop105���ϴ��ļ�
        - 4��������ݲ����⣬����������ʵ�ּ�Ⱥ����ƽ��
            - ����
            ```
                ./start-balancer.sh
            ```
- ���۾����ݽڵ�
    - ��Ӱ������ͺ�����
        - �������ͺ�������hadoop����Ⱥ������һ�ֻ��ơ�
        - ��ӵ��������������ڵ㣬���������NameNode�����ڰ������������ڵ㣬���ᱻ�˳�����ӵ��������������ڵ㣬���������NameNode����������Ǩ�ƺ��˳���
        - ����������ȷ���������NameNode��DataNode�ڵ㣬��������һ����workers�ļ�����һ�¡� �����������ڼ�Ⱥ���й���������DataNode�ڵ㡣
    - ���`������`�����裺(`����������`)
        - 1����NameNode�ڵ��/opt/module/hadoop-3.1.3/etc/hadoopĿ¼�·ֱ𴴽�whitelist ��blacklist�ļ�
            - whitelist�е�����Ϊ
            ```
            hadoop102
            hadoop103
            hadoop104
            hadoop105
            ```
            - blacklist��ʱΪ�գ�``��
        - 2����hdfs-site.xml�����ļ�������dfs.hosts�� dfs.hosts.exclude���ò���
        ```
        <!-- ������ -->
        <property>
            <name>dfs.hosts</name>
            <value>/opt/module/hadoop-3.1.3/etc/hadoop/whitelist</value>
        </property>
        <!-- ������ -->
        <property>
            <name>dfs.hosts.exclude</name>
            <value>/opt/module/hadoop-3.1.3/etc/hadoop/blacklist</value>
        </property>
        ```
        - 3���ַ������ļ�
        - 4����Ⱥ����
##

























































