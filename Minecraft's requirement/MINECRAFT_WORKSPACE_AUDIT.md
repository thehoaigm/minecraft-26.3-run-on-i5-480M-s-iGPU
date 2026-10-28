# Kiểm kê workspace Minecraft 26.3

Ngày khảo sát: 2026-10-28  
Workspace: C:\Users\theho\AppData\Roaming\PrismLauncher\instances\26.3  
Trạng thái: kiểm tra runtime/instance ở chế độ chỉ đọc; các file trong danh sách báo cáo được tạo hoặc cập nhật theo yêu cầu. Không sửa launcher setting, JAR, mods, DLL hệ thống hay driver.

## Kết luận nhanh

- **Đã xác minh:** Đây là PrismLauncher instance OneSix, cấu hình Minecraft Java 26.3 và LWJGL 3.4.3. Không phải repository source; Git báo không phải repository.
- **Đã xác minh:** Client JAR 26.3 có ở thư mục thư viện dùng chung của PrismLauncher. SHA-1 và kích thước khớp metadata 26.3. JAR là bytecode và tài nguyên đã đóng gói, không phải source đầy đủ.
- **Đã xác minh:** Instance không có mods, resource packs, shader packs, config Minecraft, world, game logs hay crash reports.
- **Đã xác minh:** Máy là Windows 10 Home 64-bit build 19045, Intel Core i5 M 460, 2 lõi/4 luồng, 4 GiB RAM và Intel HD Graphics với driver 2012. D3D12 WARP tạo được thiết bị feature level 12.1.
- **Mâu thuẫn / chưa xác minh:** Người dùng nêu native OpenGL 2.1, nhưng WGL probe trong phiên này chỉ thấy Microsoft GDI Generic OpenGL 1.1, không có GLSL. Probe không gắn được vào Intel ICD nên không chứng minh mức tối đa thật của Intel GPU. Không coi OpenGL 2.1 là đã đo.
- **Blocker:** Project C++ yêu cầu nằm trong Documents, ngoài vùng ghi cho phép của phiên này. Không tạo source trong thư mục instance để né ràng buộc.

## Inventory ban đầu và snapshot sau khi lập báo cáo

Tại inventory ban đầu có 5 file, 1 thư mục và 53.530 byte: .md gồm 2 file/37.043 byte; .txt 1/15.765 byte; .cfg 1/79 byte; .json 1/643 byte. Sau khi tạo các báo cáo, cây hiện có 12 file và 1 thư mục; nhóm đuôi file cùng dung lượng snapshot được ghi dưới đây. Thư mục `minecraft/` vẫn rỗng.

Snapshot hiện tại: .md 08 file/0000076369 byte; .txt 02 file/0000015789 byte; .cfg 01 file/0000000079 byte; .json 01 file/0000000643 byte; tổng 12 file/0000092880 byte.

- AGENTS.md — 29.882 byte, đặc tả hiện hành.
- idkprompt.txt — 15.765 byte, bản nội dung nhiệm vụ trùng lặp.
- instance.cfg — 79 byte, cấu hình PrismLauncher.
- mmc-pack.json — 643 byte, component Minecraft/LWJGL.
- PROJECT_ANALYSIS.md — báo cáo tổng hợp hiện hành, trỏ tới sáu báo cáo chi tiết.
- return prompt.txt — file text 24 byte, nội dung `codex resume --no-daemon`; nguồn gốc chưa xác minh.
- minecraft/ — thư mục rỗng.

Các báo cáo được tạo/cập nhật trong instance root: MINECRAFT_WORKSPACE_AUDIT.md, RENDERING_REQUIREMENTS.md, JAVA_WRAPPER_PLAN.md, MOD_COMPATIBILITY_MATRIX.md, CPP_REIMPLEMENTATION_PLAN.md, RENDER_TEST_RESULTS.md và PROJECT_ANALYSIS.md. Không có source C++ trong instance.

Không có .git, source, build scripts, tests, CI, patch metadata, mods/, resourcepacks/, shaderpacks/, config/, logs/, crash-reports/, screenshots/ hoặc saves/ trong instance. Git status/log không áp dụng: git báo “not a git repository”.

## Instance và metadata

- instance.cfg: InstanceType=OneSix, name=26.3.
- mmc-pack.json: component net.minecraft version 26.3; component org.lwjgl3 version 3.4.3; metadata PrismLauncher quy định component LWJGL mà instance sử dụng.
- Shared metadata: C:\Users\theho\AppData\Roaming\PrismLauncher\meta\net.minecraft\26.3.json; version=26.3, type=release, main class net.minecraft.client.main.Main, compatibleJavaName=java-runtime-epsilon, compatibleJavaMajors=25, requires org.lwjgl3 3.4.3, asset index id 34, 51 thư viện Java.
- Shared client JAR: libraries\com\mojang\minecraft\26.3\minecraft-26.3-client.jar, 41.483.720 byte. SHA-1 thực tế E877B6A07ACD633FB3BB475002175CEC036E7B87 khớp metadata e877b6a07acd633fb3bb475002175cec036e7b87.
- JAR version.json: id=26.3, build_time=2026-09-15T11:20:48Z, java_component=java-runtime-epsilon, java_version=25, world version=5023, protocol=777. Class files có major version 69, tương ứng Java 25.
- Asset index 34.json: 5.147 object; đã kiểm tra từng đường dẫn hash trong assets\objects và cả 5.147 object đều tồn tại. Tổng số byte kỳ vọng bị thiếu: 0.
- LWJGL 3.4.3 artifact đã có trong shared libraries gồm lwjgl, lwjgl-opengl, lwjgl-sdl, lwjgl-shaderc, lwjgl-spvc, lwjgl-vulkan, lwjgl-vma, cùng các native Windows x64 tương ứng cho SDL, OpenGL, ShaderC, SPVC và các thành phần khác. Đây là cache dùng chung, không nằm trong instance.

## Loader, mod, source và runtime

- mmc-pack chỉ khai báo Minecraft và LWJGL; không có component Fabric, Forge, NeoForge, Quilt hay loader khác. Không tìm thấy thư mục mods hoặc JAR mod trong instance; central mods directory không có ở root PrismLauncher.
- Không tìm thấy source Minecraft, source mod, license source để port, Gradle/CMake project hoặc tests. JAR được kiểm tra ở mức metadata, tên class, hằng số bytecode và shader resources; không decompile để tái sử dụng code.
- PrismLauncher global config có JavaPath rỗng, JavaVersion=1.8.0_51, AutomaticJavaSwitch=true và AutomaticJavaDownload=true. Runtime bundled trong Prism là jre-legacy Java 8. Có Liberica OpenJDK 21.0.4 64-bit trên máy, nhưng Minecraft 26.3 metadata yêu cầu Java 25.
- Không thấy Java 25 trong thư mục Java của Prism; game chưa chạy nên runtime Java thực tế mà Prism sẽ chọn/tải khi launch chưa xác minh. Java 8/21 không đạt major 25 theo metadata.
- PrismLauncher logs có 10 dòng lỗi tải object từ resources.download.minecraft.net (timeout/remote close/protocol). Hiện tại asset index 34 hoàn chỉnh; các lỗi log không chứng minh asset 26.3 đang thiếu.
- Không có latest.log hoặc crash reports cho game, vì instance chưa ghi log game trong đường dẫn này.

## Hệ điều hành, CPU, RAM, GPU và API

Đo bằng dxdiag, Windows APIs, CPUID và probe tạm trong %TEMP%; không ghi file chẩn đoán vào instance.

| Thành phần | Kết quả | Trạng thái |
|---|---|---|
| OS | Windows 10 Home 64-bit, build 19045 | Đã đo bằng dxdiag |
| CPU | Intel Core i5 M 460 @ 2.53 GHz; x86-64; 2 lõi / 4 luồng; SSE2/SSE3/SSE4.1, không AVX | Đã đo bằng CPUID và Windows API |
| RAM | 4.083.867.648 byte tổng; khoảng 691 MB còn trống tại thời điểm probe | Đã đo; thay đổi theo thời gian |
| GPU | Intel HD Graphics (Core i5), driver 8.15.10.2900, ngày 2012-11-26; dxdiag báo 1.696 MB display memory | Đã đo bằng dxdiag; chưa đo GPU compute/API feature set |
| D3D12 WARP | Adapter Microsoft Basic Render Driver; D3D12CreateDevice thành công; feature level tối đa đo được 12.1 | Đã đo bằng probe DXGI/D3D12 |
| Native OpenGL context | WGL tạo được Microsoft Corporation / GDI Generic / GL 1.1.0; GLSL version null; WGL_ARB_create_context không có | Đã đo trong phiên này; không phải phép đo Intel ICD |
| Vulkan loader | LoadLibrary(vulkan-1.dll) lỗi Win32 126; không có vulkaninfo; không có VK_ICD_FILENAMES/VK_DRIVER_FILES | Chưa có loader khả dụng trong tiến trình probe; ICD/device feature set chưa thể liệt kê |
| Dung lượng đĩa | Không lấy được số đáng tin qua sandbox | Chưa xác minh |

Minecraft.net công bố target minimum hiện hành là Vulkan 1.3, 8 GB RAM với GPU rời hoặc 12 GB với iGPU, CPU 4 lõi và 1080p/30 FPS Fast. Trang đó cũng nói phần cứng dưới target vẫn có thể chạy hiện tại nhưng hiệu năng/chất lượng không được bảo đảm. Máy đo được 4 GB, 2 lõi và Intel HD nên thấp hơn target công bố; đây không phải kết luận rằng client chắc chắn không khởi chạy. Nguồn: https://www.minecraft.net/en-us/article/minecraft-java-edition-system-requirements

## Thành phần ngoài instance được kiểm tra

Chỉ kiểm tra cache/runtime dùng chung mà metadata instance tham chiếu; không quét nội dung các instance khác.

- PrismLauncher shared component metadata: meta\net.minecraft\26.3.json.
- Client runtime: libraries\com\mojang\minecraft\26.3\minecraft-26.3-client.jar.
- Java/native libraries: libraries\org\lwjgl\*\3.4.3 và các native-windows artifact được metadata dùng.
- Asset index: assets\indexes\34.json và đúng các hash object mà index đó tham chiếu.
- Java runtimes: PrismLauncher\java chỉ có jre-legacy; JDK 21 hệ thống nằm ở C:\Program Files\BellSoft\LibericaJDK-21.
- Log launcher mới nhất: PrismLauncher\logs\PrismLauncher-0.log; thông tin tài khoản/định danh trong log không được đưa vào báo cáo.
- Không kiểm tra account file hoặc secret launcher.

## Chưa thể xác minh

- Native OpenGL maximum/profile/extensions thực sự của Intel HD: probe chỉ nhận GDI Generic, không nhận driver Intel.
- Vulkan ICD/API version/features thực sự trên Intel HD. Hiện tiến trình không load được Vulkan loader.
- Java runtime chính xác Prism chọn lúc launch: chưa launch, JavaPath rỗng và auto switch/download đang bật.
- Backend được chọn lần cuối: không có options.txt hoặc game log. JAR chứa cả OpenGL và Vulkan backend.
- Không có thông tin tương thích mod vì không có mod được cài.





