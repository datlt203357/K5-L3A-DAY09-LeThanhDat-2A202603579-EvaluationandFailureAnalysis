# Trace case: từ trace tới quyết định sai đầu tiên

Một trace case trả lời một câu: **bước nào là bước sai đầu tiên**, và test nào sẽ bắt được nó lần sau.

```bash
cd src
python -m lab9.trace_case --shape swarm --claim C-002 --seed 55
python -m lab9.trace_case --shape fixed --claim C-002 --seed 55   # đối chứng cùng claim/seed
```

## Case đã phân tích: swarm tin ticket thay dữ liệu order

| Trường | Nội dung |
|---|---|
| Trial | `swarm C-002@55`; kết quả lấy từ trace output bạn chạy |
| Triệu chứng | Grader FAIL: `status resolved != denied`; ledger ghi refund 20.000đ, nhãn kỳ vọng refund 0 và `denied`. |
| Bước sai đầu tiên | Event 3 `model_override`: `delay_days` đổi thành 5. |
| Vì sao là bước đầu tiên | Event 2 `investigator.lookup_order` trả dữ liệu chuẩn `delay_days=2`. Handoff và lookup đúng; sai lệch xuất hiện lần đầu ở event 3, trước quote (event 4, 20.000đ). |
| Failure mode | `FM-03 ticket_trust` — dùng số ngày trong ticket thay cho dữ liệu order. |
| Đối chứng | Chạy `fixed C-002@55` để so cùng claim/seed; kết quả expected là không override dữ liệu và refund 0. |
| Regression assertion | Với `swarm C-002@55`, grader phải fail vì `status`, `refund_vnd` và `refund_count` lệch nhãn; `fixed C-002@55` phải pass. |
| Chưa chắc chắn | Trace này cho thấy lỗi đúng ở seed 55; bốn seed còn lại của C-002 pass, nên lỗi có tính seed-dependent. |

## Kết quả bộ trace public

Trace bạn gửi có 70 trial (14 claim × 5 seed): 69 PASS, 1 FAIL là `C-002@55`. Đây là lỗi quan sát được trong `swarm`; các mã FM khác trong taxonomy là failure mode được eval set dùng để mô tả/nhắm tới, không phải kết luận rằng swarm đã mắc các lỗi đó trong lần chạy này.

`trace_case` khởi tạo store mới cho mỗi lần gọi. Vì vậy các lần `C-011@seed` và `C-012@seed` trong batch chỉ kiểm tra chạy đơn; chúng không xác minh retry hoặc crash recovery. Hai hành vi đó được kiểm bởi test đa bước riêng `test_t11_retry_keeps_one_refund` và `test_t12_crash_before_checkpoint_does_not_double_pay` trong `tests/test_public_matrix.py`.

Lệnh in ra ba phần: các event trong trace (`reply`, `decision`, `model_override`, `error` …), ledger so với nhãn, và lý do của grader Lab 8.

## Mẫu điền (mỗi nhóm 1 case, lấy từ trial đang fail trong report)

| Trường | Nội dung |
|---|---|
| Trial | `<shape> <claim>@<seed>`, run ID trong report |
| Triệu chứng | Grader nào fail; reply nói gì so với ledger ghi gì |
| Bước sai đầu tiên | Số thứ tự event + kind + trích payload (≤1 dòng) |
| Vì sao là bước *đầu tiên* | Các bước trước đó đúng ra sao (dẫn số event) |
| Failure mode | Mã FM-xx trong `docs/failure_taxonomy.md` |
| Đối chứng | Cùng claim@seed trên shape `fixed`: bước đó khác thế nào |
| Regression test | Tên test + grader/assert nào bắt được; đã thấy fail trên variant lỗi và pass trên `fixed` chưa |
| Chưa chắc chắn | Điều trace không chứng minh được |

## Quy tắc

- Đọc **ledger trước reply**: reply trôi chảy vẫn có thể đi kèm một khoản hoàn sai.
- "Bước sai đầu tiên" phải dẫn số event cụ thể, không phải "agent bị nhầm".
- Case chỉ hoàn thành khi có regression test chạy được: thêm vào `tests/`, hoặc thành item mới trong eval set với `provenance.from_trace`.
