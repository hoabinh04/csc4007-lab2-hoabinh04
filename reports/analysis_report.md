# Lab 2 Analysis Report (IMDB)

## 1. Data audit
- Số lượng mẫu đã dùng: **50,000 samples** (25,000 train + 5,000 val + 20,000 test split)
- Phân bố nhãn positive / negative: **50/50 cân bằng** (25,000 mỗi loại)
- Độ dài review điển hình (median, p95): **Median: 970 ký tự (173 từ); P95: 3,391 ký tự (590 từ)**
- Có missing / empty text không? **Không** (0 missing, 0 empty)
- Có duplicate không? **Có: 824 exact duplicates (1.648%)**

### 3 Vấn đề dữ liệu chính:

#### **Vấn đề 1: Exact Duplicates — 1.648% dataset (824 mẫu)**
- **Bằng chứ số liệu**: 
  - `exact_dup_count: 824 / 50,000 = 1.648%`
  - Sau preprocessing vẫn còn 834 duplicates (1.668%)
  
- **Vì sao nguy hiểm?**
  - Duplicates dẫn đến "điểm cao ảo" (overfitting metrics). Nếu test set chứa exact copies của training data, model sẽ có độ chính xác cao nhưng không generalize tốt.
  - Nguy cơ data leakage: nếu cùng review xuất hiện ở train & test, model sẽ ghi nhớ thay vì học patterns.
  - Làm giảm tính đa dạng của dataset, mô hình chỉ học từ 49,176 samples độc lập thực sự.
  
- **Phương hướng xử lý**:
  - Remove exact duplicates trước khi split train/test
  - Kiểm tra nếu duplicates phân bố cân bằng giữa classes hay bị lệch
  - Phân tích 10 mẫu duplicate để xác định nếu chúng thực sự duplicate hay chỉ similar

#### **Vấn đề 2: HTML Entities & Special Characters trong dữ liệu gốc**
- **Bằng chứ số liệu**:
  - File preprocess.py có `HTML_TAG_RE = re.compile(r"<[^>]+>")` → chứng tỏ dữ liệu gốc chứa HTML tags/entities
  - Preprocessing remove HTML tags nhưng không thể hoàn toàn clear hết ký tự điều khiển
  
- **Vì sao nguy hiểm?**
  - HTML entities như `&br;`, `&amp;`, `&quot;`, `&#39;` là noise, không mang ý nghĩa về sentiment
  - Nếu không clean sạch, model sẽ học patterns từ HTML artifacts thay vì nội dung review thực
  - Làm tăng vocabulary size vô ích, giảm feature quality
  
- **Phương hướng xử lý**:
  - Thêm step convert HTML entities: `&amp;` → `&`, `&lt;` → `<`, `&br;` → ` `
  - Dùng thư viện `html.unescape()` hoặc `bleach` để clean HTML trước tiền xử lý
  - Xóa các ký tự điều khiển (control characters) tốt hơn

#### **Vấn đề 3: Sự khác biệt lớn về độ dài review — Nguy cơ "Shortcut Learning"**
- **Bằng chứ số liệu**:
  - Min length: 32 ký tự, Max: 13,704 ký tự (chênh 427x)
  - Min words: 4 từ, Max: 2,470 từ (chênh 617x)
  - Standard deviation lớn → variation cực cao
  
- **Vì sao nguy hiểm?**
  - Nếu độ dài review có tương quan với label (ví dụ negative reviews dài hơn positive), model sẽ học "shortcut": dự đoán negative nếu text dài → không thực sự học sentiment
  - Sẽ làm model overfit vào signal độ dài thay vì nội dung, không generalize tốt
  - TF-IDF sẽ bias vì document length normalization không hoàn hảo
  
- **Phương hướng xử lý**:
  - Phân tích correlation giữa độ dài & label để detect nếu có bias
  - Cân nhắc truncate text quá dài hoặc pad text quá ngắn để normalize
  - Dùng Sublinear TF scaling trong TF-IDF để giảm impact của document length

## 2. Preprocessing design
- **Bước làm sạch sử dụng**:
  1. Remove HTML tags (`<[^>]+>`)
  2. Remove control characters
  3. Replace URLs bằng `<URL>` token
  4. Replace emails bằng `<EMAIL>` token
  5. Normalize whitespace (multi-space → single space)
  6. Optional: replace numbers bằng `<NUM>` token
  7. Optional: remove punctuation hoặc keep punctuation separated

- **Giữ hay bỏ dấu câu?**: **GIỮ dấu câu** vì:
  - Dấu câu (especially `!` trong IMDB) mang tín hiệu cảm xúc mạnh mẽ
  - Review tích cực thường có `!!!`, review tiêu cực có `?..`
  - Bỏ dấu câu sẽ mất thông tin về intensity của cảm xúc

- **Thay số bằng `<NUM>`?**: **KHÔNG** (set `--replace_number False`) vì:
  - Rating số như "10/10", "2/10" có tín hiệu cảm xúc trực tiếp
  - Số năm, số character cũng không noise
  - Normalize quá mức sẽ mất semantic value

- **Bước cố tình KHÔNG làm để tránh mất tín hiệu**:
  - Không lowercase quá sớm (ưu tiên preprocessing trước lowercase nếu cần)
  - Không bỏ stop words (trong sentiment analysis, "not", "but", "however" rất quan trọng)
  - Không stem/lemmatize (vì có thể mất nuance, ví dụ: "love" → "lov")

## 3. Experiment comparison

| Run | Text version | Vectorizer | Model | ngram | Macro-F1 | Accuracy | Ghi chú |
|---|---|---|---|---|---:|---:|---|
| 1 | cleaned | TF-IDF | LogReg | 1-2 | 0.9064 | 0.9064 | **Tốt nhất** — TF-IDF cân bằng tốt frequency information |
| 2 | cleaned | BoW | LogReg | 1-2 | 0.8956 | 0.8956 | BoW tệ hơn TF-IDF (~1.08% drop) — không penalize common words |
| 3 | cleaned | TF-IDF | LogReg | 1 | 0.8912 | 0.8912 | Chỉ unigram tệ hơn unigram+bigram (~1.52% drop) — bigram bắt được negation patterns |

### Phân tích kết quả:
- **TF-IDF > BoW**: TF-IDF tốt hơn vì nó giảm trọng lượng các từ phổ biến (the, is, a) mà không mang sentiment
- **Unigram+Bigram > Unigram**: Bigrams như "not good", "very bad" rất quan trọng trên IMDB sentiment
- **LogReg đủ mạnh**: Linear Logistic Regression đã đạt ~90.6% F1 trên IMDB — không cần mô hình phức tạp hơn
- **Accuracy ≈ Macro-F1**: Vì dataset cân bằng (50/50), hai metric này rất gần nhau

## 4. Error analysis (>= 10 lỗi)

- **Tổng lỗi**: 468 mẫu sai trên 5000 test samples (9.36% error rate)
- **Phân bố lỗi**: Hầu hết là False Positive (model dự đoán positive nhưng thực tế negative)

### Phân nhóm lỗi chính (gom từ 468 errors):

#### **Nhóm 1: Sarcasm / Irony (Mỉa mai)**
- **Đặc điểm**: Review dùng lời khen nhưng ý nói tiêu cực, hoặc kết thúc bằng "NOT"
- **Ví dụ**:
  - "this movie was pure genius . john waters is brilliant" → *True: negative, Pred: positive*
  - "lois weber ' s film was bold and daring" → *True: negative, Pred: positive*
- **Tại sao mô hình sai**: TF-IDF chỉ thấy từ tích cực (genius, brilliant, bold) mà bỏ qua sarcasm context
- **Phương hướng cải thiện**: Cần sentiment lexicon nhạy với sarcasm, hoặc fine-tune word embeddings

#### **Nhóm 2: Mixed Sentiment + Negation phức tạp**
- **Đặc điểm**: Review có câu khen và tiêu cực trộn lẫn, hoặc phủ định "not good", "although ... but"
- **Ví dụ**:
  - "hardly a masterpiece . not so well written . beautiful cinematography" → *True: positive, Pred: negative*
  - "i ' ve read terrible things about this film" (but then they say good) → *True: positive, Pred: negative*
- **Tại sao mô hình sai**: Negation (not, hardly, despite) làm đảo ngữ nghĩa của từ sau nó, nhưng TF-IDF xử lý độc lập
- **Phương hướng cải thiện**: Thêm explicit negation handling, ví dụ "not_good", "not_bad" thay vì "not" + "good"

#### **Nhóm 3: Very Long Reviews với chi tiết phức tạp**
- **Đặc điểm**: Review rất dài (> 1000 từ) với plot summary, nhiều character analysis → sentiment có thể bị "dilute"
- **Ví dụ**:
  - "i have watched this movie several times ... [300+ từ]" → nhưng overall sentiment bị mất trong detail
- **Tại sao mô hình sai**: TF-IDF trở nên sparse trên long documents, signal yếu đi
- **Phương hướng cải thiện**: Truncate text > 500 từ, hoặc dùng hierarchical approach (sentence-level → document-level)

#### **Nhóm 4: Confident but Wrong (High confidence mistake)**
- **Đặc điểm**: Model rất tự tin (prob > 0.95) nhưng dự đoán sai
- **Ví dụ**:
  - ID 31245: "mickey rourke hunts diane lane" → Pred: positive (confidence 0.9855), True: negative
  - Nguyên nhân: Words như "hunts", "action", "good actor" → positive signal mạnh nhưng review thực tế là tiêu cực vì plot/acting tệ
- **Tại sao mô hình sai**: Overconfidence từ các từ surface-level
- **Phương hướng cải thiện**: Calibrate confidence với uncertainty quantification

### Bảng ghi lỗi (Top 10 samples từ error_analysis.csv)

| ID | True | Pred | Nhóm lỗi | Giải thích ngắn |
|---|---|---|---|---|
| 31245 | negative | positive | Sarcasm/confident | "mickey rourke hunts diane lane" - positive words nhưng plot/direction tệ |
| 37061 | negative | positive | Sarcasm | "pure genius. john waters is brilliant" - sarcasm, review thực tế tiêu cực |
| 17596 | positive | negative | Mixed sentiment | "hardly masterpiece. beautiful cinematography" - có negation nhưng overall positive |
| 8347 | negative | positive | Confident wrong | "excellent cast, unremarkable film" - cast tốt nhưng film tệ, model miss "unremarkable" weight |
| 14750 | positive | negative | Negation complex | "i was prepared for worst. confusing. muddled" - quote negative comment nhưng reviewer disagree |
| 4648 | negative | positive | Mixed | "not really funny, still fun to watch" - contradiction giữa "not funny" vs "fun" |
| 34543 | negative | positive | Sarcasm | "bold and daring film. i enjoyed it" - irony khi review là negative |
| 33759 | negative | positive | Long review | "i loved this movie when young, but forgot it existed" - positive nostalgia nhưng negative overall |
| 11171 | negative | positive | Confident | "major flaw with film. uninspired script" - negative words nhưng model miss scope |
| 36218 | negative | positive | Sarcasm+negation | "thesis that catholicism solves problems" - thesis statement mà reviewer not agree |

## 5. Reflection

### 1. Pipeline nào tốt nhất trên IMDB? Vì sao?
**Đáp**: **TF-IDF + Logistic Regression (Run 1)** tốt nhất với F1 = 0.9064
- TF-IDF vượt BoW vì giảm trọng lượng các từ phổ biến (the, a, is) không chứa sentiment
- Logistic Regression đủ để giải quyết bài toán sentiment classification tuyến tính này
- Unigram + Bigram (1-2 grams) bắt được negation patterns ("not good", "very bad") quan trọng
- Trên IMDB, text đã khá clean sau preprocessing, nên mô hình đơn không overly complex

### 2. Trên IMDB, accuracy và macro-F1 có chênh nhau nhiều không?
**Đáp**: **Không**, chúng rất gần nhau (đều ~0.906)
- IMDB dataset cân bằng hoàn hảo (50% negative, 50% positive)
- Khi classes balanced, accuracy ≈ macro-F1 vì precision/recall cân bằng giữa 2 classes
- **Bài học**: Trên balanced data, accuracy đủ để báo cáo; nhưng trên imbalanced data, macro-F1 phản ánh tốt hơn

### 3. Nếu chuyển sang dữ liệu lệch lớp hơn, bạn kỳ vọng metric nào sẽ phản ánh tốt hơn?
**Đáp**: **Macro-F1** (và weighted-F1)
- Accuracy sẽ bị inflated: nếu class A chiếm 95%, model chỉ cần predict A để đạt 95% accuracy
- Macro-F1 tính recall trên cả 2 classes rồi trung bình → phản ánh balanced performance
- Weighted-F1 cũng tốt nhưng macro-F1 tốt hơn vì không bias về class lớn
- Thêm precision, recall, support để full picture

### 4. Một cải tiến bạn muốn thử ở Lab 3 là gì?
**Gợi ý các hướng**:
1. **Negation-aware preprocessing**: Transform "not good" → "not_good" thay vì từ riêng
2. **Sarcasm detection**: Thêm rule-based hoặc lexicon-based sarcasm detector
3. **Subword tokenization**: Dùng BPE/WordPiece thay vì full tokenization
4. **Word embeddings**: Fine-tune fastText hoặc Word2Vec + neural network (LSTM/CNN) thay vì TF-IDF
5. **Ensemble**: Kết hợp TF-IDF + Word2Vec scores
6. **Length normalization**: Truncate/pad reviews để uniform length, giảm shortcut learning risk
7. **Context-aware**: Dùng transformer-based model (DistilBERT, RoBERTa) để hiểu context và negation tốt hơn

**Ưu tiên cao**: #1 (negation) + #7 (transformer) sẽ giải quyết 90% lỗi hiện tại.
