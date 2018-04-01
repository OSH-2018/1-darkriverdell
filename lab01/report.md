
# Lab01
### ���� Ubuntu 16.04.4 LTS �����
### ����qemu, 
## ʵ�鲽�裺

### 1.�������ļ�ϵͳrootfs.img
����root��ݵ�¼,Ȼ��ִ������

~~~
git clone https://github.com/mengning/menu.git
cd menu
gcc -Wall -pthread -o init linktable.c menu.c test.c -m32 -static
cd ../rootfs 
cp ../menu/init ./ 
find . | cpio -o -Hnewc |gzip -9 > ../rootfs.img 
~~~
���ܻ���ʾȱ��ĳЩ�⣬����ѡ������multilib�⣬���߰�����ʾ����


### 2.���ز������ں�
���ص�linux�ں˰汾Ϊlinux-4.13����ѹ��ʹ�� make menuconfig �����ںˣ���kernel hacking�е�compiler�����н�compile the kernel with debug infoѡ��,Ȼ��make ���뼴��

make menuconfig ������Ҫncurses�⣬����Ҫ�������ն˿�ȴ���80�ַ���ע�ⲻҪ���ն�̫С��

### 3.����׷���ں�
��rootfs.img�ƶ����ں��ļ��У�Ȼ��װqemu
���ն�����������:

qemu -kernel arch/x86/boot/bzImage -initrd rootfs.img -s -S

qemu�������Ҷ����ڿ�ʼ״̬
����һ���ն��н���gdb
�������

file vmlinu
break start_kernel
target remote:1234

��ʱgdb����������qemu�����Դ�gdb����qemu����ͣ�ͼ���

## ��Ҫ�¼�
˵�������������ᵽ�ĺܶຯ����ֻ������4.13�汾
###  init_task�Ĵ���
Init_task��ϵͳ�ĵ�һ�����̣�Ҳ�������������̵����ȡ�������start_kernel����֮ǰ����ƽ̨��صĴ����б������ġ�

����һ��task_struct�ṹ�壨���а�������100�����ݣ�����ϵͳ��̬���壬������ͨ��fork��kernel_thread��������Ȼ��init_task����ʱϵͳ����ʵ�ϸ���˵��û�н�����һ������ʼ�Ļ����뵽start_kernel��������������������֮�С�init_task �ɺ�INIT_TASK ��������ʼ������Ҫ��˵�������������ڴ��е�λ�ã�����stack ,thread_info�����壬����һЩ״̬��Ϣ���ȵȡ�
### start_kernel������
start_kernel��ƽ̨�޹ص��ں˴������ڣ�����ʵ�ϴ��뻹������������archĿ¼��ִ�У�����һ���ǳ��Ӵ�ĺ�����������Ҫ����������ں˵ĳ�ʼ���Լ����ɵ�һ���û�̬����init������˵��ʼ����������������ڿ�����Ⱥcgroups_init_early���������ƽ��̶���Դ��ʹ�ã�������cpu����(boot_cpu_init())����ʼ���ڴ����ϵͳ(mm_init())����ʼ�������ļ�ϵͳ(vmalloc_init())����ʼ��������(sched_init())����ʼ��ҳ����ʼ���ж���������ʼ��ϵͳʱ��ȵȵȵȵȵȡ���

start_kernel������һЩ������ƵĽ�������Ϊջ���ý�˿ȸֵ����ʼ����������Ļ��ƣ���ʼ����������������ڵĻ��Ƶȵȡ�

start_kernel��������rest_init����

### init������kernel_thread���̵Ĵ���
��rest_init �����е�������kernel_thread��������kernel_init��kernel_thread��

![rest_init �������ε���kernel_thread]()

kernel_init���̲���֮��ϵͳ������ռ�ж�����Դ�Ľ��̸��kernel_init��ɸ�������֮�󣬻�ͨ������do_execve()��������û�̬����init. ��init_task���ջ��ݻ���idle���̡�����˵�������ȵ���init_idle_bootup_task����ǰ���̣�Ҳ����init_task�����ȼ����idle�ࡣ
![����init_idle_bootup_task]()

�ٵ���schedule()��������һ�£���ǰ���̾��л�����init���̡�

![����init_idle_bootup_task]()

�˺����do_idle()��init_task���̾ͳ��ױ���˹���Ϊѭ�������̶��е�idle���̡�

![����init_idle_bootup_task]()

pid = 1 �� init ���̻�������߽׵ĳ�ʼ�������ʼ�� driver, �� console, �����ݲ�ͬ��ϵͳ���ã� ִ�ж�Ӧ�ĳ�ʼ���ű� ����������һ��Ԥ��� init ����

![����init_idle_bootup_task]()

�������ϵͳ�ĳ�ʼ�������а�������shell���棬���ں��û�������

pid = 2 �� kthreadd ��һ���ں��̣߳��������������ں��̵߳ĸ��̣߳������������ں��̴߳�����������������ǹ���͵��������ں��߳�kernel_thread,����ѭ��ִ��һ��kthread�ĺ������ú����������Ǽ��kthread_create_listȫ����������Ϊ��ʱ�Ͱѵ�һ���˳��б�����kernel_thread()�����̡߳�������е��ں��̶߳���ֱ�ӻ��߼�ӵ���kthreaddΪ������



