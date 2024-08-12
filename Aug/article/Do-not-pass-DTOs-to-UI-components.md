# [Do not pass DTOs to UI components](https://darios.blog/posts/do-not-pass-dtos-to-ui-components?utm_source=newsletter.reactdigest.net&utm_medium=referral&utm_campaign=how-airbnb-smoothly-upgrades-react)

### 🗓️ 번역 날짜: 2024.08.12

### 🧚 번역한 크루: 마스터위(명재위)

---

## DTO를 UI 컴포넌트에 넘기지 마세요.

Dario Djuric
Updated On Mar 17, 2024  
Keyword : **React**

프론트엔드 개발자로서 우리는 종종 백엔드 API나 서비스에서 전달되는 데이터 전송 객체(DTO)와 함께 작업합니다. 이러한 DTO는 네트워크를 통해 전송되는 원시 데이터 구조를 나타냅니다. 그러나 DTO를 UI 컴포넌트에서 직접 사용하는 것은 유지보수성, 재사용성, 그리고 관심사의 분리와 관련된 문제를 초래할 수 있습니다.

### DTO 안티패턴

DTO를 직접 UI 컴포넌트에 props로 전달하면 UI 컴포넌트가 백엔드의 데이터 전송 구조에 강하게 결합됩니다. 이는 백엔드 데이터 모델이 변경될 때 컴포넌트 인터페이스를 발전시키거나 리팩터링하기 어렵게 만들 수 있습니다. 또한, DTO를 직접 사용하면 최소 권한 원칙이 위배되어 컴포넌트가 필요한 것보다 더 많은 데이터를 받게 됩니다. 결국 데이터 엑세스와 UI 렌더링 간의 역할 경계를 흐리게 만듭니다.

### 데이터 엑세스 레이어

데이터 엑세스 레이어(data access layer)는 백엔드 DTO를 애플리케이션 UI의 요구에 맞게 단순화된 객체 모델로 변환하는 맵핑 레이어로 생각할 수 있습니다. 이 과정에서 중첩된 객체를 평탄화하거나, 특정 속성만 선택하거나, 계산된 필드를 도출하는 등 필요한 데이터 변환을 수행할 수 있습니다.

데이터 엑세스 레이어는 전송 데이터 모델을 격리하여, 이러한 모델이 UI 컴포넌트 도메인에 유입되어 오염되는 것을 방지합니다. 컴포넌트는 자신이 맡은 역할에 맞게 형성된 객체 모델만 알면 되며, 데이터가 어떻게 전송되는지에 대한 세부 사항은 알 필요가 없습니다.

예를 들어, 다음과 같이 블로그 포스트에 대한 백앤드 DTO를 가지고 있다고 해봅시다.

```tsx
{
  id: "abc123",
  authorId: 42,
  title: "New Blog Post",
  content: "This is my new blog post...",
  metadata: {
    createdAt: "2022-01-15T08:25:00Z",
    updatedAt: "2022-01-15T08:25:00Z",
    tags: ["react", "javascript"]
  }
}
```

데이터 엑세스 레이어는 당신의 UI에 렌더링 하기위해 해당 DTO를 단순화한 Post 객체를 만들 것입니다.

```tsx
{
  id: "abc123",
  author: "Jane Doe",
  title: "New Blog Post",
  content: "This is my new blog post...",
  formattedDate: "January 15, 2022",
  tags: ["react", "javascript"]
}
```

latter 객체가 UI에 필요 없는 `authorId`와 같은 불필요한 속성을 생략하는 방식을 주목하세요. 또한 author 및 `formattedDate`와 같이 표시할 목적에 더 유용한 새 필드를 매핑하고 도출합니다.

### 추상화 경계 준수

데이터 엑세스 레이어가 DTO를 UI 친화적인 뷰 모델로 변환하면서, 우리는 UI 컴포넌트의 props와 인터페이스를 적절한 추상화 수준에서 설계할 수 있습니다. 컴포넌트 트리의 루트 근처에 있는 컨테이너 컴포넌트는 전체 페이지, 화면 또는 복잡한 기능을 나타내는 더 높은 수준의 추상화로 작업할 수 있습니다.

컴포넌트 계층 구조가 깊어질수록 더 세분화된 추상화를 도입하여 특정 UI 요구 사항에 필요한 데이터에만 집중하는 간결한 인터페이스를 사용할 수 있습니다. 이를 통해 최소 권한 원칙을 준수하여 컴포넌트에 필요한 정확한 데이터만 제공할 수 있습니다.

예를 들어, 가장 상위 레벨의 `BlogPost` 컴포넌트는 아마 title, content, metadata가 필요한 서브 컴포넌트들을 렌더링하기 위해 전체 post 데이터 모델이 필요합니다.

```tsx
// BlogPost.jsx
const BlogPost = ({ post }) => {
  return (
    <article>
      <BlogPostHeader
        title={post.title}
        formattedDate={post.formattedDate}
        tags={post.tags}
      />
      <BlogPostContent content={post.content} />
    </article>
  );
};
```

반면 `BlogPostHeader` 컴포넌트는 오직 해당 데이터의 한 부분 집합만이 필요합니다.

```tsx
// BlogPostHeader.js
const BlogPostHeader = ({ title, formattedDate, tags }) => {
  return (
    <header>
      <h1>{title}</h1>
      <time>{formattedDate}</time>
      <BlogPostTags tags={tags} />
    </header>
  );
};
```

그리고 `BlogPostTags`컴포넌트는 심지어 덜 필요로 합니다.:

```tsx
// BlogPostTags.js
const BlogPostTags = ({ tags }) => {
  return (
    <ul>
      {tags.map((tag) => (
        <li key={tag}>{tag}</li>
      ))}
    </ul>
  );
};
```

추상화 경계를 유지하고 컴포넌트 인터페이스를 신중하게 모델링함으로써 더 모듈화되고 유지 관리하기 쉬운 UI 아키텍처를 구축할 수 있습니다. 데이터 엑세스 레이어는 백엔드 데이터 모델의 변경으로부터 컴포넌트를 보호하며, 단순화된 props는 UI 요소의 재사용성과 구성성을 촉진합니다.

다음에 프런트엔드 애플리케이션 작업을 할 때는 각 컴포넌트의 인터페이스를 독립적으로 생각해보세요. 컴포넌트가 제공된 많은 데이터를 정말로 필요로 하나요? 아니면 훨씬 적은 데이터로도 작동할 수 있나요? API에서 가져온 데이터를 사용하는 방식을 살펴보세요. 네트워크를 통해 전달되는 객체는 UI의 컴포넌트보다 낮은 추상화 계층에 있으므로, 컴포넌트의 인터페이스는 이를 반영해야 합니다.
