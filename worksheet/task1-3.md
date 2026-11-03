# Tutorial 6 worksheet: serving patterns

**Name / ชื่อ:**รวิกานต์ ฉายากุล 

**Student ID / รหัสนิสิต:**6630283021

This is **individual work**: each student completes and submits their own copy, in their own fork.

*งานนี้เป็น**งานเดี่ยว** นิสิตแต่ละคนทำและส่งงานของตัวเอง โดยใช้ forked repository ของตัวเอง*

Copy result rows from `results/runs.csv` (or the load test's output) into the tables. Keep answers short: one to three sentences each. Answer in Thai or English.

ตอนที่ตอบ ให้นิสิตคัดลอกผลจาก `results/runs.csv` (หรือจาก output ของ load test) ลงในตาราง ตอบสั้นๆ ข้อละ 1–3 ประโยค จะตอบเป็นภาษาไทยหรือภาษาอังกฤษก็ได้

## Task 0: Baseline (copy the baseline results from `results/runs.csv` here / copy ผลลัพธ์ของ baseline จาก `results/runs.csv` มาวางที่นี่)

| Run | p50 ms | p95 ms | accuracy | large calls |
|---|---|---|---|---|
| t0-large | 493 | 588 | 93.5% | 100% |
| t0-small | 15 | 37 | 72.5% | 0% |

**Q0.** What does the large model buy, and what does it cost?

*large model ให้อะไรเพิ่ม และต้องแลกด้วยอะไร*

>ให้ accuracy เพิ่มขึ้นจาก 72.5% เป็น 93.5% เเต่เเลกด้วย latency ที่ช้าลงประมาณ 30 เท่า และถ้าใช้ large calls 100% แปลว่าทุกรีวิวต้องจ่ายเงิน และกิน capacity ที่มีแค่ 20 calls ต่อวินาที

## Task 1: Fallback prediction (copy the Task 1 results from `results/runs.csv` here / copy ผลลัพธ์ของ Task 1 จาก `results/runs.csv` มาวางที่นี่)

| Run | run time (s) | p50 ms | p95 ms | errors | pending | accuracy |
|---|---|---|---|---|---|---|
| t1-before | 32.8 | 231 | 410 | 22 | 0 | 92.7% |
| t1-after | 9.0 | 382 | 1008 | 0 | 25 | 94.9% |

**Q1.1** In `t1-before`, p50 can look *better* than the baseline. Why is p50 misleading here? Which column tells the real story?

*ใน `t1-before` ค่า p50 อาจดู "ดีกว่า" baseline ทำไม p50 จึงทำให้เข้าใจผิดในกรณีนี้ และ column ไหนบอกความจริง*

>มี 22 requests ที่ค้างจน client รอครบ 10 วินาทีเเล้วยกเลิกไป p50 ที่เห็นจึงคิดจากเฉพาะ request ที่สำเร็จ โดย column errors บอกส่วนนี้

**Q1.2** The hang affects only 10% of large-model calls. Why is it worse for the system than 10% of reviews being slow? Think about what a waiting request holds on to in the review service and the gateway.

*การค้างเกิดกับ calls ไปยัง large model แค่ 10% ทำไมจึงส่งผลต่อระบบแย่กว่าการที่รีวิว 10% ช้าลงเฉยๆ ลองคิดว่า request ที่รออยู่ถือทรัพยากรอะไรไว้ใน review service และ gateway*

>large model รันพร้อมกันได้เเค่ 4 ช่อง ถ้าอันไหนรันนานหรือค้าง จะมีการจองช่องนั้นไว้ตลอด ทำให้ตัวอื่นที่จะมาต่อคิวต้องมารอต่อๆกันไปเรื่อยๆ ระหว่างนั้น review service กับ gateway ก็ยังถือ connection ของ request ที่รออยู่ไว้ด้วย ปัญหาจึงลามจาก 10% ไปกระทบทั้งระบบ

**Q1.3** Pending reviews have no label. Suggest what should happen to them, and name the topology from the lecture that does it.

*รีวิวที่ pending ยังไม่มี label ควรจัดการกับรีวิวเหล่านี้อย่างไร และ topology ใดจาก lecture ที่ทำแบบนั้น*

>งานที่เหลือคือ "มาติด label ทีหลัง" ได้ โดยใช้ topology 4 Asynchronous / batch โดยให้ worker มาติด label เป็นช่วงๆเวลาไป

**Q1.4** We chose a 1-second timeout. What goes wrong if it is 0.1 s? And if it is 10 s?

*เราเลือก timeout 1 วินาที จะเกิดปัญหาอะไรถ้าตั้งเป็น 0.1 วินาที และถ้าตั้งเป็น 10 วินาที*

>ถ้าตั้ง timeout ที่ 0.1 วินาที อาจสั้นเกินไปจน request ปกติยังประมวลผลไม่เสร็จ ทำให้ fallback ไปตกใน pending บ่อยเกินไป แต่ถ้าตั้ง 10 วินาที จะทำให้ request ที่ค้างต้องรอนาน กิน resources และทำให้ระบบตอบสนองช้าลง

## Task 2: Cascade (copy the Task 2 results from `results/runs.csv` here / copy ผลลัพธ์ของ Task 2 จาก `results/runs.csv` มาวางที่นี่)

| Run | threshold | p50 ms | p95 ms | accuracy | large calls |
|---|---|---|---|---|---|
| t2-0.6 | 0.6 | 6 | 540 | 91.5% | 38% |
| t2-0.7 | 0.7 | 355 | 588 | 93.0% | 53% |
| t2-0.8 | 0.8 | 495 | 719 | 93.5% | 64% |
| t2-0.9 | 0.9 | 478 | 623 | 93.5% | 77% |

**Q2.1** Which threshold would you deploy for CampusEats, and why? Name the quality you give up.

*จะเลือก threshold ใดสำหรับ deploy ใน CampusEats เพราะอะไร และต้องยอมเสีย quality ใดไป*

>เลือก threshold 0.7 เพราะได้ accuracy 93.0% ซึ่งใกล้เคียงกับ baseline large model ที่ 93.5% แต่ลด large calls เหลือ 53% ช่วยลด latency และภาระของระบบ โดยยอมเสีย accuracy ไป 0.5% ที่ไม่เลือก 0.6 แม้จะใช้ large calls น้อยกว่า เพราะ accuracy ห่างจาก QR3 แค่ 1.5% ถ้า accuracy แกว่งลงจากผลการทดสอบเมื่อใช้กับรีวิวจริง ก็อาจหลุด 90% ได้ จึงยอมจ่าย large calls เพิ่มเพื่อแลกกับ ความมั่นใจว่า accuracy จะอยู่เหนือเกณฑ์ 90% มากขึ้น

**Q2.2** Look at p50 and p95 for your threshold. Why does the cascade help p50 much more than p95?

*ดูค่า p50 และ p95 ของ threshold ที่เลือก ทำไม cascade จึงช่วย p50 ได้มากกว่า p95 มาก*

>เพราะ request ส่วนใหญ่ที่ small model มี confidence สูงสามารถตอบได้ทันทีโดยไม่ต้องเรียก large model ทำให้ p50 ลดลงมาก แต่ request ที่ confidence ต่ำยังต้องส่งไป large model ทำให้ p95 ยังค่อนข้างสูง

**Q2.3** The cascade trusts the small model's confidence. Give one situation where that trust is misplaced, and its effect on users.

*cascade เชื่อ confidence ของ small model ยกตัวอย่างหนึ่งสถานการณ์ที่ไม่ควรเชื่อ และผลที่เกิดกับผู้ใช้*

>รีวิวที่ใช้คำประชดประชันหรือมีความหมายกำกวม small model อาจมั่นใจผิดและจัด label ผิด ทำให้ผู้ใช้ได้รับผลวิเคราะห์รีวิวที่ไม่ถูกต้อง เช่น รีวิว "Wow, arrived late. Amazing service." ซึ่งจริง ๆ แล้วลูกค้าไม่พอใจ แต่ Small Model เห็นคำว่า "Amazing" จึงตอบว่าเป็น Positive ด้วย Confidence สูง Cascade เลยไม่ส่งต่อให้ Large Model 

## Task 3: Cached predictions (copy the Task 3 results from `results/runs.csv` here / copy ผลลัพธ์ของ Task 3 จาก `results/runs.csv` มาวางที่นี่)

| Run | p50 ms | p95 ms | accuracy | large calls | cache hits |
|---|---|---|---|---|---|
| t3-cache | 5 | 564 | 93.0% | 38% | 35% |

**Q3.1** Which reviews benefit from the cache, and which never will?

*รีวิวแบบไหนได้ประโยชน์จาก cache และแบบไหนไม่มีวันได้ประโยชน์*

>รีวิวที่เคยถูกประมวลผลและมีผลลัพธ์อยู่ใน cache แล้วจะได้ประโยชน์ เพราะสามารถนำผลเดิมมาใช้ได้โดยไม่ต้องเรียก model ใหม่ ส่วนรีวิวใหม่ที่ไม่เคยถูกประมวลผลจะยังไม่ได้ประโยชน์จาก cache ในครั้งแรก โดยรีวิวที่ได้ประโยชน์คือรีวิวที่ มีข้อความซ้ำกันทุกตัวอักษร เช่น “Delicious!” เพราะคนพิมพ์ซ้ำกันบ่อย ส่วนรีวิวที่ มีข้อความยาวและมีรายละเอียดเฉพาะตัว จะไม่มีวันได้ประโยชน์ เพราะ โอกาสที่คนอื่นจะพิมพ์ข้อความเหมือนกันทุกตัวอักษรมีน้อยมาก

**Q3.2** Next week you deploy a better large model. What must happen to the cache, and how would you design for it?

*สัปดาห์หน้าจะ deploy large model ตัวใหม่ที่ดีกว่าเดิม ต้องทำอะไรกับ cache และจะออกแบบรองรับเรื่องนี้อย่างไร*

>ต้อง invalidate cache เก่าหรือแยก cache ตาม model version เพื่อไม่ให้ระบบนำผล prediction จาก model เก่ามาใช้ ควรเก็บ model version เป็นส่วนหนึ่งของ cache key เพื่อให้ model ใหม่สร้างและใช้ผลลัพธ์ชุดใหม่ได้

## Debrief

**Q4.** Name one thing you learned from these measurements that the lecture did not cover.

*ยกตัวอย่างหนึ่งสิ่งที่ได้เรียนรู้จากการวัดผลครั้งนี้ ที่ไม่มีพูดถึงใน lecture*

>ได้เรียนรู้ว่าการดูแค่ p50 อาจทำให้เข้าใจประสิทธิภาพของระบบผิดได้ เพราะไม่ได้สะท้อน request ที่ค้างหรือไม่สำเร็จ จึงควรดู errors, pending และ p95 ร่วมด้วย
