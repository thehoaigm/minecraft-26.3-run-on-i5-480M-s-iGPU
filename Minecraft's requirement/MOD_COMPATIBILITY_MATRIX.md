# Ma trận tương thích mod — Minecraft Java 26.3

Ngày khảo sát: 2026-10-28  
Phạm vi: instance PrismLauncher `26.3` được cấu hình ở root workspace.

## Kết quả inventory

**Đã xác minh:** Không tìm thấy mod JAR nào trong toàn bộ cây instance. `mmc-pack.json` chỉ khai báo `net.minecraft` 26.3 và `org.lwjgl3` 3.4.3; không có loader Fabric/Forge/NeoForge hoặc component mod. Không có thư mục `mods/` trong instance. JAR Minecraft vanilla và JAR thư viện LWJGL ở cache dùng chung không được tính là mod.

| Mod / loader | Tệp và phiên bản | Metadata mod | Rendering/native/API phụ thuộc | Kết luận cho wrapper | Độ tin cậy |
|---|---|---|---|---|---|
| Không có mod được cài trong instance | Không có mod JAR để định danh | Không áp dụng | Không có mod thực tế để kiểm tra mixin, GL call, shader injection, framebuffer hook, JNI hoặc dependency loader | Chưa có ma trận tương thích theo từng mod; hiện chỉ có thể đánh giá nền vanilla. Không được diễn giải thành “mọi mod tương thích” | Cao cho phạm vi instance đã quét; không bao gồm mod được launcher nạp từ vị trí ngoài cấu hình hoặc được thêm sau thời điểm khảo sát |

## Phạm vi đã kiểm tra

- Quét đệ quy cây instance, kể cả thư mục rỗng và hidden entries; tìm tên/đuôi `.jar`, thư mục `mods`, metadata loader và cấu hình liên quan.
- Đọc `mmc-pack.json`, `instance.cfg` và metadata PrismLauncher Minecraft 26.3 được tham chiếu.
- Kiểm tra archive Minecraft client và LWJGL chỉ để nhận diện thành phần runtime; không coi chúng là mod và không decompile/copy mã nguồn độc quyền.
- Không có game log hoặc `options.txt` nên chưa biết backend được chọn khi launch; cũng không có runtime trace của mod.

## Phạm vi chưa xác minh

- Mod nằm ngoài instance nhưng được nạp bởi launcher tùy biến hoặc cấu hình chưa hiện diện.
- Mod sẽ được cài trong tương lai; compatibility không thể suy ra từ vanilla.
- Tính tương thích của bất kỳ mod Java nào với Mesa, SwiftShader, ANGLE hoặc software renderer chưa được thử.
- Mod gameplay/data-only thường ít gọi trực tiếp graphics API hơn renderer/shader mods, nhưng đây chỉ là phân loại chung; không phải kết luận về một mod cụ thể.

## Quy trình khi có mod JAR thực tế

Với mỗi JAR, xác định metadata (`fabric.mod.json`, `META-INF/mods.toml`, `neoforge.mods.toml` hoặc metadata phù hợp), phiên bản và dependency loader; sau đó rà bytecode/resource references đến mixin renderer, LWJGL GL/Vulkan, native/JNI, shader, FBO/pass và event hooks. Ghi từng mục là đã kiểm tra tĩnh hay đã chạy. Gameplay test phải theo sau vanilla render test và thêm mod từng nhóm; mở menu không chứng minh compatibility.
