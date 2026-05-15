🚀 LicheePi Nano Embedded Linux Project (Yocto Version)
Dự án này thực hiện việc xây dựng một hệ điều hành Embedded Linux tối ưu cho board LicheePi Nano (Allwinner F1C100s) sử dụng Yocto Project (Kirkstone).
Dự án này còn là nhật ký thực hành và nghiên cứu chuyên sâu về **Hệ thống Nhúng (Embedded Systems)**. Mục tiêu lớn nhất không chỉ là tạo ra một hệ điều hành chạy được, mà là hiểu rõ *bản chất* của từng dòng code và từng tiến trình biên dịch. 

Thay vì sử dụng Buildroot để có kết quả nhanh chóng, tôi đã chọn **Yocto Project (Kirkstone)**. Việc này giúp tôi đi sâu vào cách một bản phân phối Linux công nghiệp được xây dựng, quản lý toolchain, và tối ưu hóa hệ thống cho phần cứng có tài nguyên cực kỳ eo hẹp (Chip Allwinner F1C100s với chỉ 32MB RAM tích hợp).

### 🏆 Các cột mốc đã đạt được (Milestones)
1. Xây dựng thành công Image cho giả lập QEMU và board vật lý.
2. Tối ưu hóa dung lượng toàn hệ thống (Kernel + RootFS) xuống dưới 20MB để tránh tràn RAM.
3. [Tải các bản build thành công tại mục Releases](<(https://github.com/kanie-reishi/licheepi-nano-build-yocto/releases/tag/OS)>) (Bao gồm Image và file log UART).
4. 
📂 CẤU TRÚC THƯ MỤC DỰ ÁN (PROJECT STRUCTURE)
licheepi-nano-os/

├── layers/               # Chứa các layer mã nguồn (Git Submodules)

│   ├── poky/             # Layer nền tảng của Yocto Project

│   ├── meta-openembedded # Các công thức phần mềm bổ trợ

│   └── meta-licheepinano # BSP chuyên biệt cho chip Allwinner F1C100s

├── configs/              # Lưu trữ "tinh hoa" cấu hình của dự án

│   ├── hardware/         # local.conf & bblayers.conf cho board vật lý

│   └── qemu/             # local.conf & bblayers.conf cho giả lập máy ảo

├── scripts/              # Các script bash tự động hóa setup môi trường

├── boot_log.txt          # Log UART minh chứng boot thành công trên QEMU

└── README.md             # Tài liệu hướng dẫn này

## 🛠 HƯỚNG DẪN TÁI LẬP (REPRODUCIBILITY)

Để tự tay build lại môi trường này với đúng các phiên bản mã nguồn tôi đã dùng để học:

```bash
# 1. Clone dự án kèm các submodule
git clone https://github.com/kanie-reishi/licheepi-nano-build-yocto.git
cd licheepi-nano-build-yocto
git submodule update --init --recursive

# 2. Áp dụng cấu hình cá nhân (VD: build cho máy ảo QEMU)
cp configs/qemu/local.conf.sample layers/poky/build/conf/local.conf
cp configs/qemu/bblayers.conf.sample layers/poky/build/conf/bblayers.conf

# 3. Khởi tạo môi trường Yocto và bắt đầu Build
source layers/poky/oe-init-build-env build
bitbake core-image-minimal
```
📥 HƯỚNG DẪN TẢI VÀ KIỂM TRA IMAGE

Do các file Image của hệ điều hành thường có dung lượng lớn và mang đặc tính nhị phân, chúng được lưu trữ và quản lý tại mục Releases của Repository này thay vì đẩy trực tiếp vào cây thư mục Git.

1. Cách tải Image
Truy cập vào mục Releases.

Tìm phiên bản mới nhất (ví dụ v1.0).

Tải về file nén tương ứng:

LicheePi_Hardware_Deliverable.tar.gz: Dành cho board thật.

LicheePi_QEMU_Deliverable.tar.gz: Dành cho giả lập.

2. Kiểm tra tính toàn vẹn và nội dung
Sau khi tải về máy chính (Host OS), bạn có thể kiểm tra nhanh nội dung image mà không cần flash:

Trên Linux: Sử dụng lệnh tar -tvf <tên_file>.tar.gz để xem danh sách file bên trong. Đảm bảo có đủ file .sunxi-sdimg (cho hardware) hoặc .ext4 (cho QEMU).

Trên Windows: Sử dụng 7-Zip để mở và kiểm tra dung lượng các file thành phần.

3. Flash vào phần cứng
Giải nén file .sunxi-sdimg.

Sử dụng công cụ BalenaEtcher hoặc lệnh dd trên Linux để ghi file ảnh này vào thẻ nhớ MicroSD.

Lưu ý: Hệ điều hành đã được cấu hình tự động nhận diện UART Console tại Baudrate 115200.
QUÁ TRÌNH THỰC THI
1. Yocto Build Pipeline
Thay vì biên dịch thủ công, quá trình build thông qua BitBake tuân theo luồng xử lý tự động:

Fetch & Unpack: Tải mã nguồn (Kernel, U-Boot, BusyBox) từ upstream và giải nén vào thư mục work.

Patch: Áp dụng các bản vá đặc thù cho kiến trúc ARM926EJ-S và board LicheePi Nano từ layer meta-licheepinano.

Configure & Compile: Cấu hình chéo (Cross-compile) bằng GCC Toolchain (arm-poky-linux-gnueabi) tạo ra các file nhị phân object (.o).

Package: Đóng gói các tệp nhị phân thành định dạng ipk (nhẹ hơn rpm hay deb, tối ưu cho bộ nhớ nhỏ).

Rootfs & Image Creation: Tập hợp các gói cần thiết (busybox, devmem2, trình quản lý khởi động) vào thư mục RootFS, sau đó dùng script class image_types_sunxi để kết xuất thành file Disk Image (.sunxi-sdimg và .ext4).

2. Boot Flow (Luồng khởi động của hệ thống)
Cấu trúc khởi động của LicheePi Nano trải qua 4 giai đoạn chính:

BROM (Boot ROM): Mã code cứng nạp sẵn trong vi điều khiển F1C100s. Khi cấp nguồn, BROM quét thẻ nhớ MicroSD tại một offset tĩnh cố định (8KB) để tìm mã boot tiếp theo.

SPL (Secondary Program Loader): Do SRAM nội bộ của F1C100s chỉ có 32KB, BROM nạp SPL (u-boot-sunxi-with-spl.bin). Nhiệm vụ của SPL là khởi tạo xung nhịp (Clock) và đặc biệt là khởi tạo bộ nhớ chính (32MB DDR SIP) để có không gian nạp U-Boot thực sự.

U-Boot: U-Boot đọc script boot.scr trên phân vùng FAT, nạp cấu hình phần cứng từ Device Tree (suniv-f1c100s-licheepi-nano-with-lcd.dtb) và load Kernel (zImage) vào RAM, sau đó trao lại quyền điều khiển cho Kernel.

Linux Kernel & Init: Kernel khởi tạo các trình điều khiển ngoại vi, mount phân vùng ext4 làm Root Filesystem và gọi tiến trình init (PID 1) để mở giao diện shell (UART Console).

3. Cấu trúc hệ thống & Tối ưu hóa (LicheePi Nano 32MB RAM)
File Ảnh Đĩa (.sunxi-sdimg thay vì .wic): Do đặc thù offset 8KB của BROM Allwinner, định dạng đóng gói truyền thống wic không đáp ứng được yêu cầu căn chỉnh cấp thấp. Layer cung cấp một cấu trúc Raw Image gài trực tiếp SPL vào vùng ẩn (từ byte thứ 8192), theo sau là phân vùng FAT (chứa boot script, zImage, dtb) và ext4 (chứa RootFS).

Tối ưu tài nguyên: Kích thước RootFS được khống chế dưới 10MB và Kernel dưới 7MB. Không biên dịch GUI hay các driver không cần thiết, giúp hệ điều hành hoạt động trơn tru trong giới hạn cực kỳ ngặt nghèo của 32MB SIP RAM.

4. Xử lý sự cố (Debugging) trong quá trình phát triển
Sự cố: Cạn kiệt RAM (Out of Memory - OOM Killer) khiến terminal crash và ngắt tiến trình liên kết (link) của gói gcc-cross (lto-wrapper) và createrepo_c khi build.
Phân tích & Khắc phục:

Nguyên nhân: Quá trình biên dịch chéo cho lõi ARMv5 (F1C100s) hoặc ARMv7 (QEMU) với thông số song song (Parallel Make) mặc định trên máy host nhiều nhân luồng đã vắt kiệt bộ nhớ hệ thống. Lỗi OOM khiến createrepo_c để lại file .repodata rác (lock file) trong thư mục làm việc, gây lỗi crash dây chuyền ở lần build RootFS tiếp theo.

Hành động: 1. Xóa trạng thái rác cục bộ của image: bitbake -c clean core-image-minimal.
2. Bổ sung ngay 8GB Swap ảo trên máy Host.
3. Cấu hình lại local.conf, thiết lập phanh tài nguyên: BB_NUMBER_THREADS = "4" và PARALLEL_MAKE = "-j 4" để khống chế xung đột bộ nhớ, đảm bảo tiến trình thành công ổn định.
