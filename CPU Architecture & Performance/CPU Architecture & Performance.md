	CẤU TRÚC HỆ THỐNG BỘ NHỚ ĐỆM (CPU CACHE & MEMORY)
 
2.1. Phân cấp Bộ nhớ đệm (L1, L2, L3 Cache)
Tốc độ của CPU tăng nhanh hơn rất nhiều so với tốc độ của RAM. Để giải quyết khoảng cách hiệu năng này (Memory Wall), các nhà thiết kế phần cứng đã thêm vào bộ nhớ đệm SRAM nằm ngay trên chip CPU:
L1 Cache: Tích hợp trong từng nhân CPU, chia thành L1i (Instruction) và L1d (Data). Tốc độ truy xuất nhanh nhất (1-3 clock cycles), dung lượng rất nhỏ (32KB - 64KB per core).
L2 Cache: Nằm riêng cho từng nhân, tốc độ trung bình (10-15 clock cycles), dung lượng từ 512KB - 2MB per core.
L3 Cache (LLC - Last Level Cache): Dung lượng lớn (vài chục đến hàng trăm MB), dùng chung cho tất cả các nhân trên cùng một vi mạch (Die).
2.2. Tính định vị bộ nhớ (Cache Locality)
Để Cache hoạt động hiệu quả, chương trình cần tận dụng nguyên lý Locality:
Spatial Locality (Định vị không gian): Nếu một ô nhớ được truy cập, các ô nhớ liền kề nó trong RAM cũng sẽ sớm được truy cập. Dữ liệu được nạp vào Cache theo từng dòng (Cache Line, thường là 64 bytes).
Temporal Locality (Định vị thời gian): Nếu một dữ liệu vừa được truy cập, khả năng cao nó sẽ tiếp tục được truy cập lại trong tương lai gần.
2.3. Cache Miss và Băng thông Bộ nhớ (Memory Bandwidth)
Cache Miss: Xảy ra khi CPU tìm dữ liệu trong Cache nhưng không thấy, buộc phải gọi xuống RAM. Một lần Cache Miss có thể khiến CPU bị treo idling từ 100 - 300 chu kỳ xung nhịp.
Memory Bandwidth: Giới hạn lượng dữ liệu tối đa RAM có thể truyền vào CPU trong một giây. Trong các bài toán AI (như nhân ma trận), hiệu năng thường rơi vào trạng thái Memory-bound (nghẽn băng thông bộ nhớ) trước khi đạt tới giới hạn tính toán của CPU (Compute-bound).

3. XỬ LÝ SONG SONG CẤP TẬP LỆNH (SIMD & VECTORIZATION)
3.1. SIMD (Single Instruction, Multiple Data)
SIMD cho phép CPU thực hiện cùng một phép tính (cộng, nhân, gán) trên nhiều phần tử dữ liệu cùng một lúc trong một chu kỳ xung nhịp duy nhất.
 
3.2. Tiến hóa của các tập lệnh mở rộng
SSE (Streaming SIMD Extensions): Sử dụng các thanh ghi XMM 128-bit. Tính toán đồng thời 4 số thực float32.
AVX / AVX2 (Advanced Vector Extensions): Mở rộng thanh ghi lên YMM 256-bit. Tính toán đồng thời 8 số thực float32 hoặc 16 số nguyên int16.
AVX-512: Mở rộng thanh ghi lên ZMM 512-bit. Tính toán đồng thời 16 số thực float32 hoặc 32 số nguyên int8. Rất mạnh cho tính toán ma trận AI.
3.3. Vectorization trong lập trình
Vectorization là quá trình tái cấu trúc các vòng lặp tuần tự (for loop) thành các lệnh SIMD. Điều này có thể thực hiện thông qua:
Auto-vectorization: Trình biên dịch C/C++ tự động tối ưu với flag -O3 và -mavx2.
SIMD Intrinsics: Viết trực tiếp mã hàm C-level do nhà sản xuất chip cung cấp (<immintrin.h>).
4. ĐA NHÂN, ĐIỀU PHỐI TIẾN TRÌNH VÀ KIẾN TRÚC BỘ NHỚ NÂNG CAO
4.1. Kiến trúc NUMA (Non-Uniform Memory Access)
Trong các máy chủ đa vi xử lý (Multi-socket), RAM được chia thành các Node vật lý gắn liền với từng CPU:
Truy cập Local Memory (RAM cắm trực tiếp ở CPU hiện tại): Độ trễ thấp, tốc độ tối đa.
Truy cập Remote Memory (RAM cắm ở CPU khác): Độ trễ cao do phải đi qua đường kết nối inter-connect (UPI/QPI).
Tối ưu AI: Phải đảm bảo tiến trình tính toán nằm trên CPU Node nào thì dữ liệu Tensor phải được cấp phát đúng RAM Node đó.
 
4.2. CPU Affinity (Độ gắn kết CPU)
Hệ điều hành thường có xu hướng di chuyển tiến trình/luồng giữa các nhân CPU để cân bằng tải. Điều này làm phá hỏng dữ liệu đang lưu trong L1/L2 Cache của nhân cũ. CPU Affinity cho phép lập trình viên "khóa" cố định một Thread vào một Core nhất định.
4.3. Context Switching & Hyper-Threading
Context Switching: Quá trình OS lưu trạng thái của luồng hiện tại và khôi phục trạng thái của luồng mới. Quá nhiều Thread vượt quá số nhân vật lý sẽ gây quá tải do thời gian chuyển đổi ngữ cảnh lớn hơn thời gian tính toán.
Hyper-Threading (SMT): Tạo ra 2 luồng ảo (Logical Core) trên 1 nhân vật lý. Tuy nhiên, với các tác vụ AI vắt kiệt tài nguyên SIMD/ALU, bật Hyper-Threading có thể làm tụt hiệu năng do 2 luồng ảo tranh chấp tài nguyên phần cứng. Quy tắc vàng cho AI trên CPU: 1 Thread = 1 Physical Core.
5. THỰC NGHIỆM VÀ MÃ NGUỒN MINH HỌA
5.1. Minh họa Tối ưu Cache Locality bằng Kỹ thuật Row-Major vs Column-Major (C++)
Row-Major (Lưu trữ theo hàng) và Column-Major (Lưu trữ theo cột) là hai phương pháp/quy tắc bố trí các mảng hai chiều (hoặc đa chiều) vào dải bộ nhớ tuyến tính 1D của máy tính.
Khi duyệt mảng 2D, việc duyệt theo hàng hay theo cột quyết định việc có tận dụng được Cache Line hay không.
5.1.1. Phân biệt Row-Major và Column-Major
Dưới đây là cách lưu trữ ma trận "A kích thước 2×3" :
"A=" [■("a" _"11" &"a" _"12" &"a" _"13" @"a" _"21" &"a" _"22" &"a" _"23"  )]
Row-Major (Lưu trữ ưu tiên Hàng)
Cách hoạt động: Dữ liệu được lưu liên tiếp hết hàng này rồi mới sang hàng tiếp theo.
Thứ tự lưu trong RAM: "[" "a" _"11"  "," "a" _"12"  "," "a" _"13"  "," "a" _"21"  "," "a" _"22"  "," "a" _"23"  "]" 
Ngôn ngữ sử dụng: C, C++, Python (NumPy mặc định), Rust, Pascal.
Column-Major (Lưu trữ ưu tiên Cột)
Cách hoạt động: Dữ liệu được lưu liên tiếp hết cột này rồi mới sang cột tiếp theo.
Thứ tự lưu trong RAM: "[" "a" _"11"  "," "a" _"21"  "," "a" _"12"  "," "a" _"22"  "," "a" _"13"  "," "a" _"23"  "]" 
Ngôn ngữ sử dụng: Fortran, MATLAB, R, Julia, các thư viện BLAS/LAPACK truyền thống.
5.1.2. Tại sao quy tắc này lại quan trọng? (Hiệu năng & CPU Cache)
Nắm vững Row-Major hay Column-Major quyết định trực tiếp tới tốc độ thực thi chương trình, đặc biệt là trong tính toán ma trận (AI/Machine Learning, đồ họa máy tính) thông qua Cache Locality (Tính định vị bộ nhớ):
Khi CPU đọc 1 phần tử từ RAM, nó không đọc đơn lẻ mà nạp cả một khối ô nhớ liền kề (gọi là Cache Line, thường là 64 bytes) vào CPU Cache.
Nếu bạn duyệt mảng theo đúng chiều lưu trữ của ngôn ngữ đó, phần tử tiếp theo bạn cần đã nằm sẵn trong Cache (Cache Hit), giúp CPU xử lý cực nhanh.
Nếu bạn duyệt ngược chiều, CPU phải liên tục nhảy cách quãng trong RAM, gây ra hiện tượng Cache Miss và làm chương trình chậm đi từ 5 đến 10 lần.
5.1.3. Minh họa minh chứng bằng Code (C++)

#include <iostream>
#include <vector>
#include <chrono>
const int N = 8000;
std::vector<std::vector<int>> matrix(N, std::vector<int>(N, 1));

// Tối ưu Cache: Duyệt theo hàng (Row-Major) - Spatial Locality Tốt
void row_major_access() 
{
 long long sum = 0;
 auto start = std::chrono::high_resolution_clock::now();
for (int i = 0; i < N; ++i) 
{
for (int j = 0; j < N; ++j) 
{
sum += matrix[i][j]; // Các phần tử liên tiếp nằm cùng Cache Line
        		}
   	}
auto end = std::chrono::high_resolution_clock::now();
std::chrono::duration<double> diff = end - start;
std::cout << "[Row-Major] Time: " << diff.count() << " s | Sum: " << sum << std::endl;
}
// Phá hỏng Cache: Duyệt theo cột (Column-Major) - Bị Cache Miss liên tục
void column_major_access() 
{
 long long sum = 0;
    	auto start = std::chrono::high_resolution_clock::now();
    
    	for (int j = 0; j < N; ++j) 
{
        		for (int i = 0; i < N; ++i) 
{
            		sum += matrix[i][j]; // Nhảy cách quãng N phần tử -> Cache Miss liên tục!
        		}
    	}
   	 auto end = std::chrono::high_resolution_clock::now();
    	std::chrono::duration<double> diff = end - start;
    	std::cout << "[Column-Major] Time: " << diff.count() << " s | Sum: " << sum << std::endl;
}
int main() 
{
row_major_access();
    	column_major_access();
    	return 0;
}
Kết quả: Cách duyệt Row-Major thường nhanh hơn Column-Major từ 5 đến 10 lần tùy thuộc vào dung lượng L1/L2 Cache của CPU.
5.2. Minh họa Tăng tốc bằng AVX2 SIMD Intrinsics (C++)
Cộng 2 mảng số thực 8 phần tử bằng vòng lặp thông thường so với lệnh AVX2 256-bit.
C++
#include <iostream>
#include <immintrin.h> // Thư viện AVX Intrinsics
void vector_add_avx2(float* a, float* b, float* result) 
{
// Nạp 8 số float (256-bit) từ mảng a và b vào thanh ghi AVX
   	 __m256 vec_a = _mm256_loadu_ps(a);
    	__m256 vec_b = _mm256_loadu_ps(b);
    	// Thực hiện 8 phép cộng đồng thời trong 1 chu kỳ xung nhịp
    	__m256 vec_res = _mm256_add_ps(vec_a, vec_b);
    	// Trả kết quả từ thanh ghi về bộ nhớ RAM
    	_mm256_storeu_ps(result, vec_res);
}
int main() 
{
 // Bộ nhớ aligned 32-byte giúp SIMD đạt hiệu năng tối đa
    	alignas(32) float a[8] = {1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0};
    	alignas(32) float b[8] = {8.0, 7.0, 6.0, 5.0, 4.0, 3.0, 2.0, 1.0};
    	alignas(32) float res[8];
    	vector_add_avx2(a, b, res);
    	std::cout << "Kết quả AVX2 Add: ";
    	for (int i = 0; i < 8; ++i) 
{
       		std::cout << res[i] << " "; // Output: 9 9 9 9 9 9 9 9
    	}
    std::cout << std::endl;
    return 0;
}
5.3. Minh họa Thiết lập CPU Affinity và Tối ưu NUMA trong Python (Linux Environment)
Python

import os
import psutil
import torch
def setup_cpu_optimization():
# 1. Khóa tiến trình hiện tại vào Core 0 và Core 1 (CPU Affinity)
    	pid = os.getpid()
    	process = psutil.Process(pid)
    	target_cores = [0, 1]
    	process.cpu_affinity(target_cores)
    	print(f"Đã gán thành công Tiến trình {pid} vào Cores: {process.cpu_affinity()}")
    	# 2. Cấu hình PyTorch không dùng Hyper-Threading và giới hạn số Thread
    	# Giúp giảm chi phí Context Switch và tranh chấp tài nguyên SIMD
    	torch.set_num_threads(len(target_cores))
    	torch.set_num_interop_threads(1)
    	print(f"Số lượng PyTorch Threads được thiết lập: {torch.get_num_threads()}")
if __name__ == "__main__":
setup_cpu_optimization()
    	# Thực thi nhân ma trận Tensor AI đã được giới hạn luồng
   	 x = torch.randn(2000, 2000)
    	y = torch.randn(2000, 2000)
    	z = torch.matmul(x, y)
    	print("Tính toán nhân ma trận Tensor hoàn tất thành công!")

