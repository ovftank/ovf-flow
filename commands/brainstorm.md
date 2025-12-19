---
name: brainstorm
description: Bắt đầu brainstorm!
argument-hint: [chủ đề]
allowed-tools: [AskUserQuestion, Read, Write, Edit, TodoWrite, Task, WebSearch]
examples:
    - brainstorm "tính năng mới cho app"
    - brainstorm "vấn đề customer pain points"
    - brainstorm "competitive analysis"
---

# Interactive Brainstorm

Chào mừng đến với phiên brainstorm tương tác có cấu trúc! Tôi sẽ dẫn dắt bạn qua quy trình brainstorm có cấu trúc để tạo ra những ý tưởng đột phá.

Bắt đầu với việc xác định chủ đề chính cho phiên brainstorm hôm nay.

## Complete Brainstorm Techniques (62 Techniques Available)

### COLLABORATIVE (5 techniques)

1. **Yes And Building**: Xây dựng đà qua cộng dương
2. **Brain Writing Round Robin**: Tạo ý tưởng im lặng rồi xây dựng
3. **Random Stimulation**: Dùng từ ngẫu nhiên làm chất xúc tác
4. **Role Playing**: Tạo solutions từ perspectives của multiple stakeholders
5. **Ideation Relay Race**: Xây dựng ý tưởng nhanh dưới áp lực thời gian

### CREATIVE (11 techniques)

6. **What If Scenarios**: Khám phá possibilities cực đoan
7. **Analogical Thinking**: Tìm creative solutions bằng cách vẽ parallels
8. **Reversal Inversion**: Deliberately lật problems upside-down
9. **First Principles Thinking**: Loại bỏ assumptions để rebuild từ fundamental truths
10. **Forced Relationships**: Connect unrelated concepts để spark innovative bridges
11. **Time Shifting**: Explore solutions qua different time periods
12. **Metaphor Mapping**: Dùng extended metaphors như thinking tools
13. **Cross-Pollination**: Transfer solutions từ completely different industries
14. **Concept Blending**: Merge hai hoặc more existing concepts
15. **Reverse Brainstorming**: Generate problems thay vì solutions
16. **Sensory Exploration**: Engage cả năm senses để discover solutions

### DEEP (8 techniques)

17. **Five Whys**: Drill down qua layers của causation
18. **Morphological Analysis**: Systematically explore tất cả parameter combinations
19. **Provocation Technique**: Dùng deliberately provocative statements
20. **Assumption Reversal**: Challenge và flip core assumptions
21. **Question Storming**: Generate questions trước khi seeking answers
22. **Constraint Mapping**: Identify và visualize tất cả constraints
23. **Failure Analysis**: Study successful failures để extract insights
24. **Emergent Thinking**: Allow solutions to emerge organically

### INTROSPECTIVE DELIGHT (6 techniques)

25. **Inner Child Conference**: Channel pure childhood curiosity
26. **Shadow Work Mining**: Explore what you're actively avoiding
27. **Values Archaeology**: Excavate deep personal values
28. **Future Self Interview**: Seek wisdom từ wiser future self
29. **Body Wisdom Dialogue**: Let physical sensations guide ideation
30. **Permission Giving**: Grant explicit permission để think impossible thoughts

### STRUCTURED (7 techniques)

31. **SCAMPER Method**: Systematic creativity qua bảy lenses
32. **Six Thinking Hats**: Explore problems qua sáu distinct perspectives
33. **Mind Mapping**: Visually branch ideas từ central concept
34. **Resource Constraints**: Generate innovative solutions với extreme limitations
35. **Decision Tree Mapping**: Map out tất cả possible decision paths
36. **Solution Matrix**: Create systematic grid của problem variables
37. **Trait Transfer**: Borrow attributes từ successful solutions

### THEATRICAL (6 techniques)

38. **Time Travel Talk Show**: Interview past/present/future selves
39. **Alien Anthropologist**: Examine familiar problems qua foreign eyes
40. **Dream Fusion Laboratory**: Start với impossible fantasy solutions
41. **Emotion Orchestra**: Let different emotions lead separate sessions
42. **Parallel Universe Cafe**: Explore solutions dưới alternative reality
43. **Persona Journey**: Embody different archetypes để access wisdom

### WILD (10 techniques)

44. **Chaos Engineering**: Deliberately break things để discover solutions
45. **Guerrilla Gardening Ideas**: Plant unexpected solutions trong unlikely places
46. **Pirate Code Brainstorm**: Take what works từ anywhere và remix
47. **Zombie Apocalypse Planning**: Design solutions cho extreme survival
48. **Drunk History Retelling**: Explain complex ideas với uninhibited simplicity
49. **Anti-Solution**: Generate ways để make the problem worse
50. **Quantum Superposition**: Hold multiple contradictory solutions
51. **Elemental Forces**: Imagine solutions being sculpted bởi natural elements
52. **Wild Card Techniques**: Combine unexpected elements

### BIOMIMETIC (3 techniques)

53. **Nature's Solutions**: Study how nature solves similar problems
54. **Ecosystem Thinking**: Analyze problem như ecosystem
55. **Evolutionary Pressure**: Apply evolutionary principles

### QUANTUM (3 techniques)

56. **Observer Effect**: Recognize how observing solutions changes behavior
57. **Entanglement Thinking**: Explore how solution elements might be connected
58. **Superposition Collapse**: Hold multiple potential solutions simultaneously

### CULTURAL (4 techniques)

59. **Indigenous Wisdom**: Draw upon traditional knowledge systems
60. **Fusion Cuisine**: Mix cultural approaches và perspectives
61. **Ritual Innovation**: Apply ritual design principles
62. **Mythic Frameworks**: Use myths và archetypal stories như frameworks

---

## Session Setup & Execution

Bây giờ tôi sẽ bắt đầu phiên brainstorm tương tác. Tôi cần sử dụng AskUserQuestion tool để thu thập thông tin từ bạn.

### Step 1: Session Type
Hỏi user: "🔧 Bạn muốn bắt đầu session mới hay tiếp tục session đã có?" với options:
- Start New Session: Bắt đầu phiên brainstorm hoàn toàn mới
- Continue Previous Session: Tiếp tục làm việc với session trước đó

### Step 2: Topic Definition
Nếu là session mới:
- Nếu $ARGUMENTS có nội dung, xác nhận topic với user
- Nếu $ARGUMENTS trống, hỏi: "🎯 Chủ đề chính bạn muốn brainstorm là gì?" với options:
  - Product Ideas: Ý tưởng sản phẩm mới
  - Problem Solving: Giải quyết vấn đề cụ thể
  - Process Improvement: Cải tiến quy trình
  - Custom Topic: Chủ đề tùy chỉnh

### Step 3: Goal Setting
Hỏi user: "🎯 Mục tiêu chính của phiên brainstorm này là gì?" với options:
- Generate Ideas: Tạo ra ý tưởng mới
- Solve Problem: Giải quyết vấn đề cụ thể
- Explore Options: Khám phá các lựa chọn
- Improve Solution: Cải tiến giải pháp hiện có

### Step 4: Technique Selection
Hỏi user: "🧠 Bạn muốn chọn kỹ thuật brainstorm như thế nào?" với options:
- Expert Recommendation: AI đề xuất kỹ thuật phù hợp nhất
- Browse by Category: Tự xem và chọn theo danh mục
- Random Selection: Chọn ngẫu nhiên để khám phá
- Quick Start: Bắt đầu nhanh với kỹ thuật phổ biến

Nếu Expert Recommendation:
Dựa trên topic và goals, đề xuất 3-5 techniques phù hợp nhất từ danh sách 62 techniques ở trên.

Ví dụ đề xuất cho "Product Ideas":
1. What If Scenarios: Khám phá possibilities cực đoan để vượt qua giới hạn tư duy
2. Cross-Pollination: Học hỏi từ các ngành khác nhau để tạo ý tưởng mới
3. SCAMPER Method: Systematic creativity qua 7 lenses để modify existing concepts

### Step 5: Execution
Sau khi user chọn technique:
1. Initialize TodoWrite với session tasks
2. Explain technique briefly nếu user yêu cầu
3. Start executing với structured prompts
4. Capture tất cả ý tưởng với Write tool

### Step 6: Research Integration (Optional)
Hỏi user: "🌐 Bạn có muốn tôi research thêm thông tin để làm phong phú brainstorm không?" với options:
- Market Research: Nghiên cứu thị trường liên quan
- Competitor Analysis: Phân tích đối thủ
- User Research: Tìm hiểu người dùng
- Technical Research: Nghiên cứu kỹ thuật
- Skip Research: Bắt đầu brainstorm ngay

### Step 7: Parallel Analysis (Complex Topics)
Nếu topic phức tạp, hỏi user: "🔀 Topic này khá phức tạp. Bạn có muốn tôi phân tích từ nhiều góc độ song song không?" với options:
- Yes, Multi-angle Analysis: Phân tích kỹ thuật, thị trường, người dùng đồng thời
- Sequential Analysis: Phân tích từng góc độ một cách tuần tự
- Focus on Core Problem: Tập trung vào vấn đề chính trước

### Step 8: Session Continuation
Sau khi có một số ý tưởng, hỏi user: "✅ Chúng ta đã có một số ý tưởng ban đầu! Bạn muốn làm gì tiếp theo?" với options:
- Continue Current Technique: Đi sâu hơn với kỹ thuật hiện tại
- Try New Technique: Áp dụng kỹ thuật khác cho góc nhìn mới
- Review & Cluster Ideas: Tổ chức và nhóm các ý tưởng
- Research & Validate Ideas: Nghiên cứu để validate ý tưởng
- End Session & Summarize: Kết thúc và tổng kết kết quả

### Step 9: Final Actions
Khi session sắp kết thúc, hỏi user: "📝 Phiên brainstorm sắp kết thúc! Bạn muốn tôi làm gì với kết quả?" với options:
- Generate Action Plan: Tạo kế hoạch hành động cụ thể
- Create Summary Report: Lập báo cáo tóm tắt toàn phiên
- Export to Project Doc: Xuất sang tài liệu dự án
- Schedule Follow-up: Lên lịch cho phiên tiếp theo
- Just Save Session: Chỉ lưu lại session hiện tại

## File Management

Session files sẽ được lưu với format: `brainstorm-sessions/YYYY-MM-DD-[topic].md`

Mỗi session file sẽ có structure:
```yaml
---
session_id: [uuid]
date: [current-date]
topic: '[topic]'
goal: '[goal]'
duration: '[duration]'
technique: '[selected-technique]'
status: '[in-progress/completed]'
---
```

## Emergency Interventions

Khi user bị bí ý tưởng:
Hỏi: "😵 Có vẻ bạn đang bí ý tưởng! Chọn cách giải cứu:" với options:
- Change Perspective: Đổi góc nhìn - làm như người khác
- Opposite Thinking: Nghĩ ngược lại vấn đề
- Take a Break: Nghỉ 2 phút rồi quay lại
- Use Random Prompt: Dùng prompt ngẫu nhiên
- Switch to Drawing: Vẽ thay vì viết

Khi user bị quá tải ý tưởng:
Hỏi: "🌊 Quá nhiều ý tưởng! Chọn cách sắp xếp:" với options:
- Quick Vote & Prioritize: Bầu chọn nhanh để ưu tiên
- Group Similar Ideas: Nhóm các ý tưởng tương đồng
- Focus on Top 3: Chỉ tập trung vào 3 ý tưởng tốt nhất
- Save All & Review Later: Lưu tất cả và xem lại sau

---

## Instructions for Claude

Bây giờ hãy bắt đầu thực thi phiên brainstorm bằng cách:

1. Sử dụng AskUserQuestion tool với Step 1 (Session Type)
2. Dựa trên response, tiếp tục với các steps tương ứng
3. Luôn sử dụng TodoWrite để track progress
4. Sử dụng Write tool để save session file
5. Nếu cần research, sử dụng WebSearch
6. Nếu cần parallel analysis, sử dụng Task tool

QUAN TRỌNG:
- LUÔN sử dụng AskUserQuestion cho mọi tương tác với user
- KHÔNG BAO GIỜ tự động chọn hoặc giả định response
- LUÔN track progress với TodoWrite
- LUÔN save session results

Bắt đầu ngay!
