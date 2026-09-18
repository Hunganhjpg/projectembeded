1. Linux System Programming 
   - Linux System Programming là lập trình tương tác trực tiếp với các cơ chế mà Linux cung cấp để quản lý CPU, bộ nhớ, tiến trình, luồng, file, thiết bị...VD:  printf("Hello");
  1.1. Process là gì?
- Là một chương trình đang được thực thi, VD:  ./hello (Linux sẽ tạo một process để chạy chương trình hello.)
- Linux quản lý process bằng một cấu trúc dữ liệu trong Kernel, thường gọi khái quát là process descriptor.
Process
│
├── PID 
├── Program code
├── Data
├── Heap
├── Stack
├── CPU registers
├── Program Counter
├── Open files
├── Virtual memory
├── Scheduling information
└── Security information
1.2. Thread:
- Nếu Process là một container chứa tài nguyên, thì Thread là một đơn vị thực thi bên trong process. VD:
Process
│
├── Thread 1
├── Thread 2
└── Thread 3
- Các thread trong cùng process thường chia sẻ: code,data,heap, open files. Mỗi thread có stack, registers, program counter riêng
1.3 Process vs Thread
Process
Thread
Có không gian địa chỉ riêng
Chia sẻ không gian địa chỉ của process
Tạo process tương đối nặng
Tạo thread nhẹ hơn
Memory độc lập
Chia sẻ nhiều memory
Giao tiếp thường phức tạp hơn
Giao tiếp dễ hơn qua shared memory
Crash thường không trực tiếp phá process khác
Một thread lỗi nghiêm trọng có thể làm cả process chết
Có PID riêng
Có thread ID riêng
Context switch thường tốn kém hơn
Thường nhẹ hơn

1.4. User Space vs Kernel Space
- Linux chia quyền truy cập thành các vùng chính:
User Space
────────────────────────────
Your program
C library
Application
────────────────────────────
System Call
────────────────────────────

Kernel Space

Linux Kernel
CPU management
Memory management
File system
Drivers
Network
Process management
1.5. Kernel Space: Kernel là phần lõi của Linux.
1.6. System call là cơ chế để chương trình User Space yêu cầu Kernel thực hiện một dịch vụ.
7. fork() — tạo Process
Khi gọi fork(), hệ điều hành sẽ nhân bản (copy) nguyên xi tiến trình hiện tại thành một tiến trình con hoàn toàn độc lập (có PID mới). Đặc điểm : Bộ nhớ hoàn toàn tách biệt: Nếu tiến trình con sửa biến x = 10, biến x ở tiến trình cha vẫn giữ nguyên giá trị cũ. An toàn tuyệt đối: Nếu tiến trình con bị lỗi nặng và sập (Crash), tiến trình cha vẫn sống bình thường. Nhược điểm: Tốn tài nguyên RAM và CPU của máy vì hệ điều hành phải sao chép bảng trang bộ nhớ; việc trao đổi dữ liệu giữa hai tiến trình phải qua IPC nên tốc độ chậm hơn.
1.8. exec() — thay thế chương trình đang chạy 
Nó thay thế code/data của process hiện tại bằng chương trình mới.
1.9. pthread_create() — tạo Thread
Thread (thường gọi là Lightweight Process - LWP) là đơn vị thực thi nhỏ nhất được bộ điều phối (Scheduler) của hệ điều hành gán thời gian sử dụng CPU.
Mối quan hệ với Process: Một Thread không thể tồn tại độc lập; nó luôn thuộc về chính xác một Process. Một Process có thể sở hữu một hoặc nhiều Thread cùng chạy đồng thời. Mô hình chia sẻ và cô lập trong Thread: Các thành phần dùng chung (Shared): Tất cả các Thread trong cùng một Process chia sẻ chung không gian địa chỉ ảo, bao gồm vùng Code, vùng Data/BSS, vùng Heap, và bảng File Descriptors (các kết nối mạng Socket, file mở). Các thành phần độc lập (Private/Per-thread): Mỗi Thread sở hữu riêng: Một vùng nhớ Stack riêng (để chứa biến cục bộ khi gọi hàm của luồng đó). Một tập Thanh ghi CPU riêng (Program Counter để biết luồng đang thực thi tới lệnh nào, Stack Pointer). Mã định danh luồng riêng (TID - Thread ID). Thread là một luồng chạy nhẹ nằm ngay bên trong một Process. Một chương trình khi bật lên mặc định đã có sẵn 1 luồng chính (Main Thread). Cách tạo Thread mới trong C: Bạn dùng hàm pthread_create() (nằm trong thư viện <pthread.h>). Hàm này sẽ giao cho một hàm con chạy song song với luồng chính. Đặc điểm : Dùng chung bộ nhớ (Shared Memory Space): Tất cả các Thread trong cùng 1 Process đều thấy và sửa được các biến toàn cục (Global Variables). Khởi tạo cực nhanh: Tốn rất ít RAM và CPU để tạo mới một Thread so với tạo một Process. Rủi ro chí mạng: Nếu 1 Thread bị sập (ví dụ lỗi truy cập vùng nhớ cấm - Segmentation Fault), toàn bộ Process và tất cả các Thread khác đều chết theo ngay lập tức. Cơ chế chuyển đổi ngữ cảnh (Context Switching) Khi CPU chuyển từ việc chạy đơn vị này sang đơn vị khác: Giữa 2 Process khác nhau: Hệ điều hành phải lưu lại toàn bộ trạng thái thanh ghi của Process 1 vào PCB của nó, chuyển đổi con trỏ bảng trang bộ nhớ ảo (CR3 trên x86), làm mới bộ nhớ đệm dịch địa chỉ (TLB Cache Flush), rồi nạp PCB của Process 2. Quá trình này tiêu tốn rất nhiều chu kỳ xung nhịp CPU. Giữa 2 Thread cùng 1 Process: Hệ điều hành chỉ cần lưu lại các thanh ghi cơ bản và con trỏ ngăn xếp sang Thread Control Block (TCB) rồi nạp thanh ghi của Thread mới. Bộ nhớ ảo và bộ đệm TLB được giữ nguyên, giúp tốc độ chuyển đổi nhanh hơn đáng kể. Phân định nơi làm việc của Process và Thread qua 2 tầng 1. Tầng User Space (Nơi mã nguồn chạy): Thực thi logic ứng dụng: Các phép tính toán, vòng lặp, xử lý dữ liệu, vẽ giao diện người dùng (GUI).
Vùng nhớ ứng dụng: Nơi chứa mã lệnh (Text), các biến toàn cục (Data/BSS), bộ nhớ cấp phát động (Heap) và ngăn xếp hàm (Stack). Mức đặc quyền: CPU hoạt động ở chế độ không đặc quyền (User Mode / Ring 3). Tại đây, mã của bạn bị cách ly tuyệt đối, không được phép can thiệp trực tiếp vào phần cứng (RAM vật lý, cổng mạng, CPU registers). 2. Tầng Kernel Space (Nơi quản lý và điều phối): Cấu trúc quản lý: Nhân Linux lưu trữ và duy trì cấu trúc dữ liệu của tiến trình/luồng (chính là task_struct - Process Control Block). Bộ điều phối (CPU Scheduler): Quyết định luồng/tiến trình nào được nạp lên CPU chạy, luồng nào phải tạm dừng hoặc chờ đợi. Quản lý tài nguyên vật lý: Cấp phát trang nhớ RAM thật (thông qua bảng phân trang Page Table), ánh xạ phần cứng, quản lý hàng đợi gửi/nhận dữ liệu mạng. Mức đặc quyền: CPU hoạt động ở chế độ đặc quyền cao nhất (Kernel Mode / Ring 0), có toàn quyền can thiệp vào phần cứng máy tính. Cầu nối chuyển đổi giữa hai tầng: System Call (Lời gọi hệ thống) Tiến trình chuyển qua lại giữa User Space và Kernel Space liên tục thông qua cơ chế System Call: Ở User Space: Tiến trình của bạn chạy các dòng code thông thường (ví dụ: tính toán dữ liệu cảm biến). Yêu cầu tài nguyên: Khi tiến trình cần một thao tác liên quan đến phần cứng (như đọc dữ liệu từ Socket TCP, cấp thêm RAM bằng malloc(), hoặc tạo luồng mới bằng pthread_create()), nó không thể tự làm. Chuyển tầng (Context Transition): Chương trình phát một lệnh ngắt mềm (System Call Trap). CPU lập tức chuyển quyền từ User Mode (Ring 3) sang Kernel Mode (Ring 0). Ở Kernel Space: Nhân Linux tiếp nhận yêu cầu, kiểm tra tính hợp lệ, thực thi lệnh (đọc dữ liệu từ card mạng Wi-Fi, ghi vào bộ đệm của kernel). Quay về User Space: Sau khi hoàn thành, nhân Linux trả kết quả và CPU chuyển ngược về User Mode để tiến trình tiếp tục chạy code của bạn.
1.10. Thread Lifecycle
pthread_create()
│
▼
Created
│
▼
Runnable
│
▼
Scheduler
│
▼
Running
/     \
/       \
sleep     blocked
\       /
\     /
▼   ▼
Runnable
│
▼
Finished
1.11. CPU Scheduling
CPU Scheduling là cơ chế của Linux Kernel dùng để quyết định process hoặc thread nào được CPU thực thi tại một thời điểm nhằm tối ưu hiệu năng và khả năng đáp ứng của hệ thống.
a. Context Switching
Là quá trình CPU chuyển từ việc thực thi một process/thread sang process/thread khác.
Thông tin cần lưu/phục hồi:
Program Counter (PC) 
CPU Registers 
Stack Pointer 
Process State 
Ý nghĩa:
Cho phép nhiều chương trình chạy đồng thời. 
Tạo cảm giác đa nhiệm trên một CPU. 
Nhược điểm:
Tốn thời gian lưu/phục hồi trạng thái. 
Quá nhiều context switch làm giảm hiệu năng.
b. Priority
Priority là mức độ ưu tiên của process/thread khi tranh chấp CPU.
Linux thường ưu tiên:
Real-time tasks 
Interactive tasks 
High-priority processes 
Lợi ích:
Đảm bảo các tác vụ quan trọng được thực hiện nhanh hơn. 
Tăng khả năng đáp ứng hệ thống.
c. CPU Affinity
CPU Affinity cho phép gán process hoặc thread chạy trên một hoặc nhiều CPU Core xác định.
Mục đích:
Giảm CPU migration. 
Tăng hiệu quả sử dụng cache. 
Cải thiện hiệu năng trong hệ thống đa lõi.
1.12. wait() — Parent chờ Child
pid_t pid = fork();
if (pid == 0)
{
    printf("Child\n");
}
else
{
    wait(NULL);
    printf("Parent\n");
}
Có hai lý do quan trọng.
1. Đồng bộ Parent–Child
Parent có thể chờ Child hoàn thành.
2. Tránh Zombie Process
Khi child kết thúc nhưng parent chưa thu nhận trạng thái kết thúc của child, child có thể tồn tại ở trạng thái zombie.
1.13. Stack
Stack dùng cho:
local variables 
function arguments 
return address 
function call frames 
Ví dụ:
1.14. Heap
Heap thường được sử dụng khi cấp phát động: int *p = malloc(sizeof(int));
phù hợp với dữ liệu có lifetime động.
1.15. mmap()
mmap() cho phép map một vùng memory vào virtual address space của process.
1.16. Virtual Memory
Đây là một trong những khái niệm quan trọng nhất của Linux.
Process tưởng rằng nó có một không gian memory riêng:
Process không trực tiếp làm việc với physical address theo cách thông thường.
Kernel + MMU quản lý việc mapping.
