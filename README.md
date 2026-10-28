<p align="center">
  <img src="Docs/assets/372dragon-banner.svg" alt="372 dragon — TLBB x64 client, server and development tools" width="100%">
</p>

<h1 align="center">372 dragon</h1>

<p align="center">
  <strong>Mã nguồn client TLBB x64, server Skynet/Lua và công cụ phát triển123.</strong><br>
  Đọc mã · Hiểu hệ thống · Build · Kiểm thử
</p>

<p align="center">
  <a href="Docs/guide/getting-started.md">Bắt đầu</a> ·
  <a href="DATA_DOWNLOAD.md">Tải data</a> ·
  <a href="Docs/guide/build-client.md">Build client</a> ·
  <a href="Docs/guide/server-setup.md">Dựng server</a> ·
  <a href="Docs/guide/README.md">Tài liệu</a> ·
  <a href="LICENSING.md">Giấy phép</a>
</p>

---

**372 dragon** là dự án phát triển client Windows x64
từ nền mã TLBB 0.85, dùng Fairy/OGRE, Direct3D 9, CEGUI và Lua, kết nối với server
Skynet/Lua trên Linux hoặc WSL2. Repository tập hợp source client/server, công cụ,
kiểm thử và tài liệu kỹ thuật để nghiên cứu, tiếp tục phát triển và kiểm tra hành vi.

> **Trạng thái — 15/10/2026:** bản xuất source phục vụ nghiên cứu và phát triển.
> Data kiểm thử có liên kết tải riêng. Chưa nghiệm thu build/cài mới đầy đủ trên
> máy độc lập. Xem [trạng thái và giới hạn](Docs/guide/status.md) trước khi dựng môi trường.

## Chọn điểm bắt đầu

| Bạn muốn làm gì? | Đọc ở đâu? |
|---|---|
| Hiểu dự án và lấy đúng các thành phần | [Bắt đầu với 372 dragon](Docs/guide/getting-started.md) |
| Tải data, kiểm hash và giải nén | [Data kiểm thử trên Google Drive](DATA_DOWNLOAD.md) |
| Dựng client Windows x64 | [Toolchain, SDK và lệnh build](Docs/guide/build-client.md) |
| Chuẩn bị server Linux/WSL2 | [Source, cấu hình riêng và dịch vụ](Docs/guide/server-setup.md) |
| Tìm vị trí code để sửa | [Kiến trúc](Docs/guide/architecture.md) · [Bản đồ repository](Docs/guide/repository-map.md) |
| Chẩn đoán lỗi cài/build/runtime | [Xử lý lỗi thường gặp](Docs/guide/troubleshooting.md) |
| Gửi bản sửa hoặc báo lỗi | [Đóng góp](CONTRIBUTING.md) · [Hỗ trợ](SUPPORT.md) |

## Có gì trong dự án?

| Thành phần | Nội dung |
|---|---|
| **Client** | C/C++, Windows x64, Fairy/OGRE, Direct3D 9; CEGUI và Lua cho giao diện |
| **Server** | Skynet/Lua, module native C/C++, logic gameplay và cấu hình mẫu |
| **Công cụ** | Build, xử lý tài nguyên, giao thức, staging và kiểm tra |
| **Tư liệu kỹ thuật** | Registry, đặc tả, ghi chép kiểm toán và bằng chứng theo mốc phát triển |
| **Data kiểm thử** | Tài nguyên client, overlay và bảng/scene server; tải riêng để giữ repository nhẹ |

## Tải source và data

| Gói | Nơi lấy | Dung lượng tham khảo |
|---|---|---:|
| Source client/server và tài liệu | Repository này; Git clone hoặc **Code → Download ZIP** | Khoảng **192 MiB** trước nén |
| Data kiểm thử | [Google Drive — cy-test-data-2026-10-15.zip](https://drive.google.com/file/d/1FG-BSvarnmCeezwr4PqqC0zHMf7syia4/view?usp=sharing) | **12,39 GB** tải về · **24,12 GB** trước nén |
| SDK và runtime bên thứ ba | [Nguồn phụ thuộc và điều khoản](THIRD_PARTY.md) | Tùy thành phần |

Gói data có **126.212 file dữ liệu**. Tên ZIP cũ được giữ để khớp file đã chia sẻ và
checksum. Data không chứa Game.exe, DLL runtime, SDK, database người chơi hay tài khoản.
Hướng dẫn kiểm SHA-256 nằm trong [DATA_DOWNLOAD.md](DATA_DOWNLOAD.md).

```powershell
git clone https://github.com/ts4m97/372dragon.git C:\Dev\372dragon
cd C:\Dev\372dragon
```

1. Lấy source vào thư mục mới, ví dụ `C:\Dev\372dragon`.
2. Đọc [yêu cầu và bố cục cài đặt](Docs/guide/getting-started.md); tải data nếu cần chạy thử.
3. Chuẩn bị phụ thuộc và [build client](Docs/guide/build-client.md).
4. Chuẩn bị [server riêng](Docs/guide/server-setup.md), cấu hình và tài khoản mới.
5. Hoàn thành [bài kiểm tra cài mới](Docs/guide/status.md) trước khi coi môi trường đã hoạt động.

## Cấu trúc repository

```text
372dragon/
├── Src/              Client, engine, common và source thư viện
├── Server/           Skynet, gameplay và cấu hình mẫu
├── Tools/            Build, pipeline tài nguyên và công cụ dự án
├── Tests/            Kiểm thử, validator và probe
├── Registry/         ID, giao thức, ABI và dấu vết nhập nguồn
├── Docs/guide/       Hướng dẫn dành cho người dùng repository
├── Docs/assets/      Banner và nhận diện 372 dragon
├── Docs/             Tài liệu kỹ thuật và lịch sử phát triển
├── Publication/      Manifest, kiểm tra data và chuẩn bị cấu hình
├── LICENSES/         Văn bản giấy phép gốc
└── .github/          Mẫu issue và pull request
```

Sau khi giải nén data, tài nguyên được bổ sung ở `Assets/CanonicalCY/Data`,
`Play/Overlay`, `Server/Game_2` và `Server/configs`. Client và server có quy trình
build riêng. [Sơ đồ kiến trúc và luồng dữ liệu →](Docs/guide/architecture.md)

## Những điều cần biết trước khi build

- **Client:** MSVC 2022, Windows SDK, CMake ≥ 3.25, Ninja, Python 3; FMOD Core 2.02.35,
  NVIDIA Cg và DirectX theo [hướng dẫn build](Docs/guide/build-client.md).
- **Server:** Linux/WSL2, toolchain C/C++, MongoDB **4.4.31**, Mosquitto; một số phụ thuộc
  native và đường dẫn Makefile còn cần đối chiếu trên máy mới.
- **Tài nguyên:** giữ nguyên tên file và encoding. Một số bảng, khóa tài nguyên và
  nội dung `all.*` sử dụng GBK; đổi encoding hàng loạt có thể làm hỏng việc nạp data.
- **Bản chạy:** quy trình dựng đầy đủ `Play` chưa được nghiệm thu trên máy sạch.
  Công cụ staging lịch sử cần được đọc và điều chỉnh trước khi dùng.

## Giấy phép và ghi công

**PolyForm Noncommercial 1.0.0** áp dụng cho các đóng góp nguyên gốc mà dự án có quyền
cấp phép, theo [phạm vi trong LICENSE](LICENSE). Code kế thừa và thư viện giữ giấy
phép, copyright và notice riêng; các phần chịu LGPL/GPL tiếp tục theo điều khoản đó.
Đây là dự án **source-available** với phạm vi giấy phép hỗn hợp.

Data kiểm thử thuộc các tác giả/chủ sở hữu tương ứng và nằm ngoài giấy phép phần mềm
của dự án. Data phục vụ đối chiếu và kiểm tra mã nguồn, bản build và khả năng tương thích.
[Thông báo data](DATA_NOTICE.md) nêu điều kiện cung cấp theo hiện trạng và giới hạn
trách nhiệm trong phạm vi pháp luật cho phép.

[Chi tiết giấy phép](LICENSING.md) · [Thành phần bên thứ ba](THIRD_PARTY.md) ·
[Nguồn data](Publication/asset-provenance.json) · [Phạm vi đóng góp](Publication/license-scope.json)

## Đóng góp và hỗ trợ

Các đóng góp hữu ích gồm cải thiện build trên máy mới, tài liệu, kiểm thử có thể tái hiện,
sửa lỗi client/server và bổ sung nguồn gốc phụ thuộc. Đọc [CONTRIBUTING.md](CONTRIBUTING.md)
trước khi gửi pull request. Khi báo lỗi, chọn mẫu trong tab **Issues** và gửi phiên bản,
bước tái hiện, hành vi mong đợi cùng log đã loại thông tin riêng.

[Hỗ trợ](SUPPORT.md) · [Bảo mật](SECURITY.md) · [Thay đổi bản công bố](CHANGELOG.md)

<details>
<summary><strong>English overview</strong></summary>

372 dragon is a TLBB 0.85-derived Windows x64
client with a separate Skynet/Lua server. The client uses Fairy/OGRE, Direct3D 9,
CEGUI and Lua. This repository contains a source snapshot, development tools,
tests and technical documentation. Test resources are distributed separately.

A complete clean-machine build and installation has not yet been validated.
Original project contributions use PolyForm Noncommercial 1.0.0 only within the
documented scope; inherited code, third-party libraries and test data retain their
respective rights and terms. The main documentation is currently in Vietnamese.

</details>
