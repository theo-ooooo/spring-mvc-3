## 🖼️ 스프링 MVC 웹 페이지 만들기

스프링 MVC와 타임리프(Thymeleaf)를 사용해 웹 화면을 구현하며, 상품 등록/조회/수정 등의 기본 기능을 직접 만들어보았습니다. 실전 웹 개발에 필요한 MVC 패턴의 실질적 적용과 흐름을 경험할 수 있었습니다.

---

### ✅ 상품 도메인 및 저장소

```java
@Data
public class Item {
    private Long id;
    private String itemName;
    private Integer price;
    private Integer quantity;
}
```

```java
@Repository
public class ItemRepository {
    private static final Map<Long, Item> store = new HashMap<>();
    private static long sequence = 0L;

    public Item save(Item item) {
        item.setId(++sequence);
        store.put(item.getId(), item);
        return item;
    }

    public Item findById(Long id) {
        return store.get(id);
    }

    public List<Item> findAll() {
        return new ArrayList<>(store.values());
    }

    public void update(Long id, Item updateParam) {
        Item findItem = findById(id);
        findItem.setItemName(updateParam.getItemName());
        findItem.setPrice(updateParam.getPrice());
        findItem.setQuantity(updateParam.getQuantity());
    }
}
```

---

### ✅ 상품 등록 - 폼 & 저장

```java
@GetMapping("/add")
public String addForm() {
    return "items/addForm";
}

@PostMapping("/add")
public String save(@ModelAttribute Item item, RedirectAttributes redirectAttributes) {
    Item savedItem = itemRepository.save(item);
    redirectAttributes.addAttribute("itemId", savedItem.getId());
    redirectAttributes.addAttribute("status", true);
    return "redirect:/items/{itemId}";
}
```

- 등록 후 PRG(Post/Redirect/Get) 패턴 사용
- URL에 `itemId`, `status`를 쿼리 파라미터로 전달하여 새로고침 시 중복 등록 방지

---

### ✅ PRG (Post/Redirect/Get) 패턴이란?

폼 제출 후 바로 뷰를 리턴하지 않고 **리다이렉트**를 수행하여 **새로고침 시 중복 요청 문제를 방지**하는 패턴입니다.

#### 잘못된 방식 (중복 등록 문제 발생)

```java
@PostMapping("/add")
public String save(@ModelAttribute Item item) {
    itemRepository.save(item);
    return "items/item"; // 새로고침 시 POST 재실행됨
}
```

#### 올바른 방식 (PRG 적용)

```java
@PostMapping("/add")
public String save(@ModelAttribute Item item, RedirectAttributes redirectAttributes) {
    Item savedItem = itemRepository.save(item);
    redirectAttributes.addAttribute("itemId", savedItem.getId());
    redirectAttributes.addAttribute("status", true);
    return "redirect:/items/{itemId}";
}
```

---

### ✅ 상품 상세 조회

```java
@GetMapping("/{itemId}")
public String item(@PathVariable Long itemId, Model model, @RequestParam(required = false) boolean status) {
    Item item = itemRepository.findById(itemId);
    model.addAttribute("item", item);
    model.addAttribute("status", status); // 등록 성공 여부 확인용
    return "items/item";
}
```

---

### ✅ 상품 목록 페이지

```java
@GetMapping
public String items(Model model) {
    List<Item> items = itemRepository.findAll();
    model.addAttribute("items", items);
    return "items/list";
}
```

---

### ✅ 상품 수정

```java
@GetMapping("/{itemId}/edit")
public String editForm(@PathVariable Long itemId, Model model) {
    Item item = itemRepository.findById(itemId);
    model.addAttribute("item", item);
    return "items/editForm";
}

@PostMapping("/{itemId}/edit")
public String edit(@PathVariable Long itemId, @ModelAttribute Item item) {
    itemRepository.update(itemId, item);
    return "redirect:/items/{itemId}";
}
```

---

### ✅ 타임리프 템플릿 예시

```html
<!-- templates/items/item.html -->
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<body>
<h1 th:text="${item.itemName}">상품명</h1>
<p>가격: <span th:text="${item.price}">10000</span></p>
<p>수량: <span th:text="${item.quantity}">10</span></p>

<div th:if="${status}">
    <p style="color:green;">상품이 성공적으로 등록되었습니다.</p>
</div>
</body>
</html>
```

---

이 섹션에서는 실전 웹 기능 구현과 함께 PRG 패턴, 리다이렉트 처리, 타임리프 뷰 연결 등 실무에서 자주 사용하는 흐름을 연습할 수 있었습니다.
