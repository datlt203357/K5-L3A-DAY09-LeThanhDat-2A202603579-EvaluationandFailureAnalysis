# Judge audit (M2)

Chạy lại từ `src/` bằng:

```bash
python -m lab9.judge --csv data/eval/judge_tone_public.csv --audit 5
python -m lab9.judge --csv data/eval/judge_helpful_public.csv --audit 5
```

FAIL là lớp dương tính. Floors hiệu chuẩn: κ ≥ 0.6, precision ≥ 0.8, recall ≥ 0.7. Rogan–Gladen dùng sensitivity = recall và specificity = TN/(TN+FP).

| Tiêu chí | n | TP / FP / FN / TN | κ | Precision | Recall | Fail observed → corrected | Đạt floors? | Quyết định |
|---|---:|---:|---:|---:|---:|---:|---|---|
| `tone_ok` (`judge_tone_public.csv`) | 100 | 24 / 4 / 6 / 66 | 0.757 | 0.857 | 0.800 | 0.280 → 0.300 | Có | Dùng để sàng lọc tone trong phạm vi rubric này; không thay thế grader an toàn/nghiệp vụ. |
| `helpful` (`judge_helpful_public.csv`) | 100 | 9 / 0 / 21 / 70 | 0.375 | 1.000 | 0.300 | 0.090 → 0.300 | **Không** | Loại khỏi gate; bỏ sót quá nhiều lỗi, đặc biệt reply báo sai tiền. |

## Audit bất đồng thủ công

Năm case đầu dưới đây đều là false negative của helpful judge: judge ghi PASS, human ghi FAIL. Ghi chú CSV cung cấp số tiền ledger để đối chiếu với reply; các nhãn human này được xác nhận đúng.

| # | Case | Judge nói | Người nói | Ai đúng? | Ghi chú và sửa rubric |
|---:|---|---|---|---|---|
| 1 | HL-015 | PASS | FAIL — sai tiền | Human | Reply báo 55.000đ, note ghi ledger 40.000đ. Bất kỳ lệch số tiền nào so với ledger phải là FAIL; judge cần được cung cấp số đã commit hoặc để code grader so sánh. |
| 2 | HL-016 | PASS | FAIL — sai tiền | Human | Reply báo 55.000đ, ledger 35.000đ. Lời xin lỗi dài không bù được thông tin giao dịch sai; rubric phải ưu tiên tính đúng sự thật hơn độ lịch sự/độ dài. |
| 3 | HL-020 | PASS | FAIL — sai tiền | Human | Reply báo 90.000đ, ledger 120.000đ. Rubric phải bắt cả báo thiếu lẫn báo thừa tiền. |
| 4 | HL-028 | PASS | FAIL — sai tiền | Human | Reply báo 50.000đ, ledger 35.000đ. So sánh amount có cấu trúc thay vì đánh giá câu trả lời theo cảm giác hữu ích. |
| 5 | HL-035 | PASS | FAIL — sai tiền | Human | Reply báo 50.000đ, ledger 35.000đ. Cùng lỗi: judge đánh PASS dù con số không khớp; dùng grader code cho điều kiện này. |

## Kết luận

- `tone_ok` đạt cả ba floors trên bộ replay: κ=0.757, P=0.857, R=0.800. Có thể dùng làm tín hiệu phụ cho tiêu chí tone, nhưng không dùng nó để quyết định approval, tiền hoặc rò dữ liệu.
- Loại `helpful` khỏi gate. P=1.0 chỉ nói rằng các FAIL do judge phát hiện đều trùng human; R=0.3 nghĩa là judge bỏ lỡ 21 trong 30 lỗi. 21 false negative cũng kéo κ xuống 0.375, thấp hơn floor.
- Sửa rubric/thiết kế: judge helpful chỉ chấm mức trả lời đúng nhu cầu và bước tiếp theo; cung cấp dữ kiện cần thiết; mọi assertion về trạng thái và amount phải được chấm bằng code đối chiếu nhãn/ledger. Không cho điểm PASS chỉ vì reply lịch sự, dài hoặc trôi chảy.
- Rogan–Gladen hiệu chỉnh fail rate từ 0.28 lên 0.30 cho tone và từ 0.09 lên 0.30 cho helpful. Đây là ước lượng tỷ lệ lỗi tổng thể, không sửa được các false negative theo từng item và không biến helpful thành gate đạt chuẩn.
- Rủi ro còn lại: đây là CSV replay công khai, không phải tập held-out độc lập. Cần thu thập nhãn human mới, kiểm tra bất đồng theo slice và kiểm tra lại rubric trước khi dùng judge trên dữ liệu hoặc bản agent mới.
