    1. Lý thuyết nền tảng (Foundations)
    Trong môi trường đa luồng (Multi-threading) hoặc đa tiến trình (Multi-processing), các luồng chia sẻ chung không gian bộ nhớ. Nếu không được kiểm soát, việc truy cập đồng thời sẽ gây ra các lỗi hành vi phức tạp.

    ritical Section (Miền găng / Vùng găng): Là đoạn mã nguồn có thao tác truy cập hoặc chỉnh sửa tài nguyên dùng chung (Shared Resource - như biến toàn cục, file, bộ nhớ). Tại một thời điểm, chỉ nên có tối đa một luồng được phép thực thi trong miền găng.

    Race Condition (Điều kiện đua): Xảy ra khi nhiều luồng cùng truy cập và thay đổi tài nguyên dùng chung một cách đồng thời, làm cho kết quả cuối cùng phụ thuộc vào thứ tự thực thi (timing/interleaving) không thể đoán trước của Hệ điều hành.

    Cách kiểm tra thành công miền găng: Chạy chương trình đa luồng với 1.000.000 lần tăng biến đếm toàn cục; nếu kết quả thu được luôn chính xác 1.000.000 thì miền găng đã được bảo vệ thành công.
2. Các công cụ đồng bộ hóa phổ biến (Synchronization Primitives)
    1. Mutex (Mutual Exclusion)
    Mutex là cơ chế khóa độc quyền cơ bản nhất trong lập trình đa luồng. Một Mutex hoạt động giống như một chiếc chìa khóa duy nhất cho một phòng kín: tại một thời điểm, chỉ có duy nhất một luồng được phép nắm giữ khóa để đi vào vùng găng (Critical Section). Điều đặc biệt của Mutex là tính chủ sở hữu (ownership) — chỉ chính luồng đã thực hiện khóa (lock) mới có quyền thực hiện mở khóa (unlock). Tất cả các luồng khác đến sau sẽ rơi vào trạng thái chờ (Blocked/Sleep) cho đến khi tài nguyên được giải phóng.
        Kịch bản sử dụng: Mutex phù hợp nhất cho các thao tác bảo vệ tài nguyên độc quyền như: cập nhật số dư tài khoản ngân hàng, ghi file log, sửa đổi biến toàn cục hoặc thực hiện các câu lệnh cập nhật cơ sở dữ liệu.
        // Biên dịch trên Ubuntu: g++ mutex_demo.cpp -o mutex_demo -pthread
#include <iostream>
#include <thread>
#include <mutex>
#include <vector>

int counter = 0; // Tài nguyên dùng chung (Shared Resource)
std::mutex mtx;  // Công cụ bảo vệ vùng găng

void increment(int thread_id) {
    for (int i = 0; i < 10000; ++i) {
        // std::lock_guard giúp lock khi tạo và tự động unlock khi thoát khỏi scope (RAII)
        std::lock_guard<std::mutex> lock(mtx);
        counter++; 
    }
}

int main() {
    std::vector<std::thread> threads;

    // Tạo 5 luồng cùng chạy hàm increment
    for (int i = 0; i < 5; ++i) {
        threads.push_back(std::thread(increment, i));
    }

    for (auto& th : threads) {
        th.join();
    }

    std::cout << "[Mutex] Ket qua cuoi cung cua counter: " << counter << std::endl;
    std::cout << "(Ket qua ky vong: 50000)" << std::endl;
    return 0;
}
    2. SemaphoreSemaphore là công cụ đồng bộ hóa dựa trên một biến đếm nguyên (Integer variable S) dùng để quản lý việc truy cập đồng thời vào tài nguyên có số lượng giới hạn. Có hai thao tác chính trên Semaphore là wait() (giảm biến đếm S) và signal() (tăng biến đếm S). Khi giá trị S > 0, các luồng có thể đi qua và giảm S đi 1. Khi S = 0, bất kỳ luồng nào gọi wait() sẽ phải dừng lại và chờ cho đến khi có một luồng khác gọi signal() để trả lại tài nguyên. Khác với Mutex, Semaphore không có tính chủ sở hữu; bất kỳ luồng nào cũng có thể gọi tín hiệu giải phóng tài nguyên.
        Kịch bản sử dụng: Semaphore cực kỳ hiệu quả trong việc quản lý tập hợp tài nguyên giới hạn (Resource Pooling), chẳng hạn như quản lý số lượng kết nối tối đa đến Database (Database Connection Pool) hoặc giới hạn tần suất gửi yêu cầu trong hệ thống (Rate Limiting).
        // Biên dịch trên Ubuntu: g++ semaphore_demo.cpp -o semaphore_demo -pthread
#include <iostream>
#include <thread>
#include <semaphore> // Yêu cầu C++20
#include <vector>
#include <chrono>

// Giới hạn tối đa 3 luồng được truy cập cùng lúc
std::counting_semaphore<3> pool(3);

void worker(int id) {
    std::cout << "Luong " << id << " dang cho ket noi...\n";
    
    pool.acquire(); // Giảm biến đếm semaphore (Wait)
    
    std::cout << "-> Luong " << id << " DA LAY DUOC KET NOI (Dang xu ly...)\n";
    std::this_thread::sleep_for(std::chrono::milliseconds(1000));
    std::cout << "<- Luong " << id << " hoan thanh, tra ket noi.\n";
    
    pool.release(); // Tăng biến đếm semaphore (Signal)
}

int main() {
    std::vector<std::thread> threads;

    // Tạo 6 luồng cùng xin cấp phát kết nối
    for (int i = 1; i <= 6; ++i) {
        threads.push_back(std::thread(worker, i));
    }

    for (auto& th : threads) {
        th.join();
    }

    return 0;
}
    3. Spinlock
    Spinlock là một dạng khóa đồng bộ đặc biệt không đưa luồng rơi vào trạng thái ngủ (Sleep) khi bị từ chối truy cập. Thay vào đó, luồng chờ sẽ chạy một vòng lặp liên tục để liên tục kiểm tra xem khóa đã được mở ra hay chưa — cơ chế này gọi là Busy-Waiting (chờ bận). Ưu điểm của Spinlock là tránh được chi phí chuyển đổi ngữ cảnh (Context Switch Cost) của Hệ điều hành. Tuy nhiên, nhược điểm là nó sẽ chiếm dụng 100% tài nguyên CPU của core đang chạy vòng lặp đó trong suốt khoảng thời gian chờ đợi.
        Kịch bản sử dụng: Spinlock chỉ nên được áp dụng khi thời gian thực thi trong vùng găng cực kỳ ngắn (thường trong lập trình nhân hệ điều hành - Kernel Development), nơi thời gian chờ khóa ngắn hơn rất nhiều so với thời gian để CPU thực hiện một lần Context Switch.
    4. Read-Write Lock (RW Lock)
    Read-Write Lock (RW Lock) được thiết kế nhằm tối ưu hóa hiệu năng bằng cách phân biệt rõ ràng giữa thao tác đọc (Read) và thao tác ghi (Write) trên cùng một tài nguyên. RW Lock áp dụng quy tắc: Cho phép nhiều luồng cùng đọc đồng thời (Multiple Readers), nhưng chỉ cho phép duy nhất một luồng ghi tại một thời điểm (Single Writer). Khi một luồng đang thực hiện ghi, tất cả các luồng đọc và ghi khác đều phải chờ. Ngược lại, khi các luồng chỉ đọc dữ liệu, chúng không triệt tiêu quyền truy cập của nhau.
        Kịch bản sử dụng: Công cụ này lý tưởng cho các ứng dụng có tần suất đọc dữ liệu rất cao nhưng tần suất sửa đổi lại cực kỳ thấp, ví dụ như hệ thống bộ nhớ đệm (Cache), bảng định tuyến mạng, hoặc danh sách cấu hình hệ thống.
        // Biên dịch trên Ubuntu: g++ rwlock_demo.cpp -o rwlock_demo -pthread
#include <iostream>
#include <thread>
#include <shared_mutex> // Yêu cầu C++17
#include <vector>
#include <chrono>

int shared_data = 100;
std::shared_mutex rw_mtx;

// Thao tác ĐỌC: Sử dụng shared_lock (Nhiều luồng vào cùng lúc)
void reader(int id) {
    std::shared_lock<std::shared_mutex> lock(rw_mtx);
    std::cout << "Reader " << id << " doc gia tri: " << shared_data << std::endl;
    std::this_thread::sleep_for(std::chrono::milliseconds(100));
}

// Thao tác GHI: Sử dụng unique_lock (Độc quyền hoàn toàn)
void writer(int id, int new_val) {
    std::unique_lock<std::shared_mutex> lock(rw_mtx);
    std::cout << "=== Writer " << id << " CAP NHAT gia tri thanh: " << new_val << " ===\n";
    shared_data = new_val;
    std::this_thread::sleep_for(std::chrono::milliseconds(200));
}

int main() {
    std::vector<std::thread> threads;

    // Chạy xen kẽ giữa các luồng đọc và ghi
    threads.push_back(std::thread(reader, 1));
    threads.push_back(std::thread(reader, 2));
    threads.push_back(std::thread(writer, 1, 200));
    threads.push_back(std::thread(reader, 3));

    for (auto& th : threads) {
        th.join();
    }

    return 0;
}
    5. Condition Variable (CV)
    Condition Variable (CV) là công cụ cho phép các luồng giao tiếp và thông báo trạng thái cho nhau dựa trên việc kiểm tra một điều kiện logic thực tế. Thay vì một luồng phải liên tục lặp lại kiểm tra điều kiện (gây lãng phí CPU), nó sẽ tự đưa mình vào trạng thái tạm dừng (Wait) và nhả Mutex ra. Khi một luồng khác làm thay đổi trạng thái và thỏa mãn điều kiện, luồng đó sẽ gửi một tín hiệu (Signal cho 1 luồng hoặc Broadcast cho tất cả các luồng) để đánh thức các luồng đang chờ tiếp tục công việc.
        Kịch bản sử dụng: Condition Variable là thành phần cốt lõi để triển khai mô hình bài toán kinh điển Producer - Consumer (Người sản xuất - Người tiêu thụ), bài toán hàng đợi thông điệp (Message Queue), hoặc xử lý các sự kiện bất đồng bộ giữa các luồng.
        // Biên dịch trên Ubuntu: g++ cv_demo.cpp -o cv_demo -pthread
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>
#include <queue>
#include <chrono>

std::queue<int> buffer;
std::mutex mtx;
std::condition_variable cv;
bool finished = false;

// Luồng Producer: Tạo sản phẩm
void producer() {
    for (int i = 1; i <= 5; ++i) {
        std::this_thread::sleep_for(std::chrono::milliseconds(500)); // Giả lập sản xuất
        {
            std::lock_guard<std::mutex> lock(mtx);
            buffer.push(i);
            std::cout << "[Producer] Da tao san pham: " << i << std::endl;
        }
        cv.notify_one(); // Thông báo cho Consumer
    }

    {
        std::lock_guard<std::mutex> lock(mtx);
        finished = true;
    }
    cv.notify_all();
}

// Luồng Consumer: Tiêu thụ sản phẩm
void consumer() {
    while (true) {
        std::unique_lock<std::mutex> lock(mtx);
        
        // Chờ đến khi buffer có dữ liệu HOẶC producer đã xong
        cv.wait(lock, [] { return !buffer.empty() || finished; });

        while (!buffer.empty()) {
            int val = buffer.front();
            buffer.pop();
            std::cout << "  -> [Consumer] Da xu ly san pham: " << val << std::endl;
        }

        if (finished) break;
    }
}

int main() {
    std::thread t1(producer);
    std::thread t2(consumer);

    t1.join();
    t2.join();

    std::cout << "Xử lý hàng đợi hoàn tất!\n";
    return 0;
}
    6. Barrier
    Barrier (Rào chắn) là điểm tập kết đồng bộ hóa dùng cho một tập hợp gồm nhiều luồng phối hợp cùng làm việc. Khi các luồng thực thi chạy đến điểm Barrier, chúng sẽ dừng lại và đứng chờ tại đó. Chương trình chỉ cho phép tất cả các luồng cùng lúc vượt qua điểm Barrier để tiếp tục công việc tiếp theo khi và chỉ khi toàn bộ số lượng luồng đăng ký đã cùng đến được điểm tập kết này.      
        Kịch bản sử dụng: Barrier rất phổ biến trong các ứng dụng tính toán song song hiệu năng cao (High-Performance Computing - HPC), tính toán đồ họa, hoặc các thuật toán chia để trị (Divide and Conquer), nơi việc tính toán được chia thành nhiều giai đoạn (phases) và giai đoạn sau bắt buộc phải chờ kết quả hoàn tất của toàn bộ các luồng ở giai đoạn trước.
        // Biên dịch trên Ubuntu: g++ barrier_demo.cpp -o barrier_demo -pthread -std=c++20
#include <iostream>
#include <thread>
#include <barrier> // Yêu cầu C++20
#include <vector>
#include <chrono>

// Định nghĩa điểm tập kết cho 3 luồng
std::barrier sync_point(3, []() noexcept {
    std::cout << "\n=== [BARRIER] Tat ca luong da den diem tap ket! Chuyen sang giai doan tiep theo. ===\n\n";
});

void task(int id) {
    // Giai đoạn 1
    std::cout << "Luong " << id << " dang chay Giai doan 1...\n";
    std::this_thread::sleep_for(std::chrono::milliseconds(100 * id)); // Thời gian xong khác nhau
    
    std::cout << "Luong " << id << " hoan thanh GD1, dang cho cac luong khac tai Barrier...\n";
    sync_point.arrive_and_wait(); // Chờ đủ 3 luồng

    // Giai đoạn 2
    std::cout << "Luong " << id << " bat dau chay Giai doan 2.\n";
}

int main() {
    std::vector<std::thread> threads;

    for (int i = 1; i <= 3; ++i) {
        threads.push_back(std::thread(task, i));
    }

    for (auto& th : threads) {
        th.join();
    }

    return 0;
}
3. Các vấn đề kinh điển & Giải pháp (Concurrency Issues)
Khi áp dụng đồng bộ hóa, việc thiết kế sai logic có thể gây ra hiện tượng nghẽn hoặc đóng băng hệ thống.

A. Deadlock (Bế tắc)
Xảy ra khi hai hoặc nhiều luồng bị tắc nghẽn vĩnh viễn vì mỗi luồng đều đang chờ tài nguyên mà luồng kia đang nắm giữ.

4 điều kiện cần để xảy ra Deadlock (Coffman Conditions)
Deadlock chỉ xảy ra khi đồng thời thỏa mãn 4 điều kiện: Mutual Exclusion (Loại trừ lẫn nhau), Hold and Wait (Giữ và chờ), No Preemption (Không tranh đoạt), và Circular Wait (Chờ vòng tròn).

Giải pháp: Đánh số thứ tự tài nguyên và bắt buộc tất cả các luồng phải khóa tài nguyên theo đúng thứ tự đó (Resource Hierarchy), hoặc sử dụng cơ chế try_lock có giới hạn thời gian (Timeout).

B. Livelock vs. Starvation vs. Priority Inversion
Livelock (Bế tắc động): Các luồng liên tục thay đổi trạng thái để nhường nhau nhưng không luồng nào tiến triển được công việc (tương tự hai người đi ngược chiều trong ngõ hẹp liên tục tránh về cùng một phía).

Starvation (Đói tài nguyên): Một hoặc nhiều luồng liên tục bị bỏ qua, không bao giờ được cấp phát tài nguyên do hệ thống ưu tiên các luồng khác.

Priority Inversion (Đảo ngược độ ưu tiên): Luồng có độ ưu tiên cao (High) bị hoãn thực thi do phải chờ một khóa đang bị nắm giữ bởi luồng có độ ưu tiên thấp (Low).

Giải pháp: Sử dụng cơ chế Priority Inheritance (Luồng Low tạm thời được nâng độ ưu tiên lên bằng luồng High cho đến khi nhả khóa).