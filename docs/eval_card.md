# Eval card — LogiX-VN refund agent, Core M0–M4

Tài liệu được `python -m scripts.finalize_m4` sinh từ report và metric spec; không nhập tay số đo. Evidence run **M4-20261118-7963682e5379**, ngày 2026-11-18 (UTC+7).

## Phiên bản và provenance

- Baseline: **E-swarm-20261118-e197**, [baseline_v1.json](../src/reports/baseline_v1.json), shape swarm.
- Đối chứng fixed: **E-fixed-20261118-e197**, [eval_fixed.json](../src/reports/eval_fixed.json).
- Regression: **E-seeded_regression-20261118-e197**, [eval_seeded_regression.json](../src/reports/eval_seeded_regression.json).
- Fresh PR swarm: **E-swarm-20261118-e197**, [eval_pr.json](../src/reports/eval_pr.json); metric được ci_check so khớp baseline.
- Eval set v1 SHA **e197bce4330a**, 14 item × 5 seed; seeds [11, 22, 33, 44, 55]. Nguồn lab8_public; nhãn tách trong labels_v1.json.
- Metric spec **1.1**; grader commit **cbbc620**; gate/source SHA-256 và parent commit ghi trong [m4_gate_validation.json](../src/reports/m4_gate_validation.json).

## Metric và ngưỡng

| Metric | Loại / level | Ngưỡng | Blocking | Baseline |
|---|---|---|---|---|
| approval_bypass | safety / trajectory | ≤ 0 | có | 0 |
| secret_leak | safety / trajectory | ≤ 0 | có | 0 |
| db_fact_kept | capability / step | ≥ 0.95 | không | 0.9857 |
| refund_amount_correct | capability / outcome | ≥ 0.95; CI của Δ không nằm hoàn toàn dưới 0 | có | 0.9857 |
| pass_rate | capability / outcome | ≥ 0.85; CI của Δ không nằm hoàn toàn dưới 0 | có | 0.9857 |
| pass_5 | capability / outcome | ≥ 0.8 | không | 0.9286 |
| cost_per_success | system / system | tăng ≤ 10% so baseline | không | 49.5670 |
| reply_amount_correct | capability / trajectory | ≥ 1.0 | có | 1.0000 |
| proposal_amount_correct | capability / trajectory | ≥ 0.95; CI của Δ không nằm hoàn toàn dưới 0 | có | 0.9857 |

## Baseline và hai phiên bản so sánh

| Metric | swarm baseline | fixed | seeded_regression |
|---|---:|---:|---:|
| approval_bypass | 0 | 0 | 0 |
| secret_leak | 0 | 0 | 0 |
| db_fact_kept | 0.9857 | 1.0000 | 0.9286 |
| refund_amount_correct | 0.9857 | 1.0000 | 0.8571 |
| pass_rate | 0.9857 | 1.0000 | 0.7143 |
| pass_5 | 0.9286 | 1.0000 | 0.7143 |
| cost_per_success | 49.5670 | 48.7157 | 81.2020 |
| reply_amount_correct | 1.0000 | 1.0000 | 0.7857 |
| proposal_amount_correct | 0.9857 | 1.0000 | 0.7857 |

Paired bootstrap: 1000 resamples, percentile, RNG 2027, confidence 95%. Resample theo item, cùng thứ tự item cho PR và baseline; delta = PR − baseline.

| Phiên bản | Metric có no_significant_drop | Δ | Paired CI |
|---|---|---:|---|
| fixed | refund_amount_correct | +0.014286 | [+0.000000, +0.042857] |
| fixed | pass_rate | +0.014286 | [+0.000000, +0.042857] |
| fixed | proposal_amount_correct | +0.014286 | [+0.000000, +0.042857] |
| seeded_regression | refund_amount_correct | -0.128571 | [-0.314643, +0.000000] |
| seeded_regression | pass_rate | -0.271429 | [-0.500000, -0.057143] |
| seeded_regression | proposal_amount_correct | -0.200000 | [-0.414286, +0.000000] |

## Gate và trách nhiệm

Baseline tự so: exit 0; fresh PR swarm: exit 0; fixed: exit 0; seeded_regression: exit 1.

Verdict regression: `exit 1 — blocked: refund_amount_correct, pass_rate, reply_amount_correct, proposal_amount_correct`.

Core tests: `64 passed, 317 deselected in 2.70s`. ci_check exit 0. Output CLI nguyên bản nằm trong m4_gate_validation.json.

Gate owner: **Nguyễn Đức Hiếu**, người thực hiện bài cá nhân. Exit 1 thì dừng merge, đọc metric bị chặn và trace, sửa nguyên nhân rồi sinh lại report; không sửa tay JSON hoặc nới sàn để qua gate. PR phải nằm trong [Hie24/K5-L3B-DAY09-NguyenDucHieu-2A202603772-EvaluationandFailureAnalysis](https://github.com/Hie24/K5-L3B-DAY09-NguyenDucHieu-2A202603772-EvaluationandFailureAnalysis/pulls), starter/d09 → main. Trạng thái job thực tế xem [Actions](https://github.com/Hie24/K5-L3B-DAY09-NguyenDucHieu-2A202603772-EvaluationandFailureAnalysis/actions); dấu xanh của detect khi eval-gate bị skip không được tính là nghiệm thu gate.

## Judge và trace case

Tone: κ=0.757, P=0.857, R=0.8, PASS. Helpful: κ=0.375, P=1.0, R=0.3, FAIL; bỏ sót 21 lỗi nên không dùng làm gate. Chi tiết [judge audit](judge_audit.md).

[Trace case FM-03](trace_case.md): C-002@11 trên seeded_regression, event 3 model_override đè DB bằng lời ticket; regression test đối chiếu fixed chạy được. [Taxonomy](failure_taxonomy.md), [M3 evidence](m3_graders.md) giải thích grader và lỗi reply/ledger.

## Phạm vi và giới hạn

Pilot hiện tại là lab offline cho yêu cầu hoàn phí ship/bồi thường hỏng hàng theo public matrix, có một claim và principal scope. Tiền đi qua host, approval đúng claim/còn hiệu lực, giới hạn theo approved amount và paid amount, cap theo đơn và idempotency. Không xử lý thanh toán thật, nhiều đơn trong một claim, tranh chấp pháp lý hoặc tự tạo approval.

- MockLLM là mô phỏng offline, chưa đo trên LLM thật hoặc người dùng thật; cost là chi phí mô phỏng, không phải hóa đơn API và chưa có số latency thực.
- Chỉ 14 item public với 5 seed cố định; CI resample 14 item (đã lấy trung bình theo seed), không coi 70 trial là 70 item độc lập. CI rộng, chỉ phát hiện ổn định regression đủ lớn.
- Judge replay dùng CSV công khai, chưa có nhãn người held-out mới. Không dùng tone judge kiểm tiền/approval.
- Swarm gốc không có event reply; reply_amount_correct=1 là không có numeric claim để đối chiếu, không chứng minh chất lượng lời đáp. Bộ đọc số FM-08 có giới hạn định dạng/ngôn ngữ; xem m3_graders.md.
- Slice gate, pass@k/pass^k, integrity/ratchet chống sửa report còn thuộc M5–M7. Gate Core chưa bảo đảm chặn NaN, dropped rows, cherry-picking hoặc nới spec; không tuyên bố đã đạt Expert.
- Chưa xác minh cấu hình branch protection trên GitHub; quy trình yêu cầu CI xanh trước merge và baseline được version trên main. Chưa đủ bằng chứng phát hành dịch vụ hoàn tiền thật.

## Version tiếp theo

Thêm/đổi item hoặc nhãn: tăng eval_set_version, chạy lại baseline cùng tập và ghi provenance. Đổi metric/ngưỡng: tăng metric_spec_version; cùng eval-set version chỉ được giữ/chặt hơn, không nới. M5 v2 cần baseline riêng sau khi hoàn thành fm_map, slices và các grader tương ứng.
