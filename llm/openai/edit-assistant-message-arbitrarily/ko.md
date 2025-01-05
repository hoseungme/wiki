# OpenAI assistant message를 임의로 수정하면 생기는 문제

결론 부터 말하면, OpenAI의 chat completion을 사용할 때, assistant message를 임의로 수정하는 것은 하지 않는 것이 좋다.

예를 들어, 제품에서 아래의 플로우를 개발한다고 가정해보자.

1. 유저에게 직업 입력받기
2. 그 직업과 관련있는 웹사이트 3가지를 리스트로 보여주기
3. 유저에게 리스트에서 필요한건 새로 추가하고, 필요 없는건 지우라고 하기
4. 웹사이트 리스트를 그룹으로 묶고, 그룹의 제목을 자동으로 만들어주기

그리고 1, 2단계를 위해 OpenAI로 아래와 같은 chat을 구성했다고 가정해보자. (실제로 사용한 프롬프트는 아니고, 이해를 돕기 위해 지어낸 것이니 따라하지는 말도록 하자)

```
System Prompt:
당신은 인간의 직업에 대해 모르는 것이 없는 유능한 웹 브라우저입니다.

당신은 유저의 현재 직업을 입력으로 받습니다. 유저의 직업과 가장 관련된 웹 사이트 도메인 3가지를 출력하세요.

예를 들어, 유저의 입력이 "개발자"라면, 당신은 아래와 같이 출력할 수 있습니다:
github.com
stackoverflow.com
leetcode.com
```

```
User:
개발자
```

그리고 assistant에게 아래와 같은 응답을 받았다고 해보자.

```
Assistant:
github.com
stackoverflow.com
leetcode.com
```

이때, 우리가 3, 4단계를 완성하기 위해 할 수 있는 두 가지 방법이 있다. 첫 번째는 assistant message를 수정하는 것이다.

OpenAI chat completion API를 보면, 모델(assistant)이 지금까지 주고받은 메시지를 기억하고 우리와 대화하는 구조가 아니다. 오히려 우리가 지금까지 주고받은 모든 메시지를 전부 보내줘야 하는 구조이다.

이 말은 즉, 아래와 같이 실제로 주고받지 않은 대화를 개발자가 지어낼 수가 있다는 뜻이다.

```typescript
await openAI.chat.completions.create({
  messages: [
    { role: "user", content: "안녕하세요." },
    { role: "assistant", content: "입 좀 다무세요." },
    { role: "user", content: "네..?" },
  ],
  model: "gpt-4o-mini",
  temperature: 1.0,
});
```

따라서 우리가 처음엔 github, stackoverflow, leetcode를 응답으로 받았더라도,

```typescript
{
  role: "assistant",
  content: `
  github.com
  stackoverflow.com
  leetcode.com
  `,
}
```

3단계에서 유저가 웹사이트 리스트 수정을 마치면, 애초에 모델이 해당 리스트를 응답했던 것으로 지어낼 수가 있는 것이다.

```typescript
{
  role: "assistant",
  content: `
  github.com
  www.figma.com
  developer.mozilla.org
  `,
}
```

거기에 아래와 같이 그룹의 제목을 추천해달라고 요구하는 User message를 하나 더 추가해주면 4단계까지 완성이 가능하다.

```typescript
await openAI.chat.completions.create({
  messages: [
    {
      role: "system",
      content: `
      당신은 인간의 직업에 대해 모르는 것이 없는 유능한 웹 브라우저입니다.

      당신은 유저의 현재 직업을 입력으로 받습니다. 유저의 직업과 가장 관련된 웹 사이트 도메인 3가지를 출력하세요.

      예를 들어, 유저의 입력이 "개발자"라면, 당신은 아래와 같이 출력할 수 있습니다:
      github.com
      stackoverflow.com
      leetcode.com
      `,
    },
    { role: "user", content: "개발자" },
    {
      role: "assistant",
      content: `
      github.com
      www.figma.com
      developer.mozilla.org
      `,
    },
    {
      role: "user",
      content: `
      위 웹사이트들을 하나의 그룹으로 묶을 건데, 그 그룹의 제목을 만드세요.
      `,
    },
  ],
  model: "gpt-4o-mini",
  temperature: 1.0,
});
```

두 번째 방법은 user message에 최대한 많은 컨텍스트를 제공하는 것이다. 모델에게 모델이 준 리스트를 수정했다는 사실, 그리고 수정된 결과물까지 말해주는 것이다.

```typescript
await openAI.chat.completions.create({
  messages: [
    {
      role: "system",
      content: `
      당신은 인간의 직업에 대해 모르는 것이 없는 유능한 웹 브라우저입니다.

      당신은 유저의 현재 직업을 입력으로 받습니다. 유저의 직업과 가장 관련된 웹 사이트 도메인 3가지를 출력하세요.

      예를 들어, 유저의 입력이 "개발자"라면, 당신은 아래와 같이 출력할 수 있습니다:
      github.com
      stackoverflow.com
      leetcode.com
      `,
    },
    { role: "user", content: "개발자" },
    {
      role: "assistant",
      content: `
      github.com
      stackoverflow.com
      leetcode.com
      `,
    },
    {
      role: "user",
      content: `
      당신이 제공한 웹사이트 목록을 아래와 같이 수정했어요:
      github.com
      www.figma.com
      developer.mozilla.org

      이제 위 웹사이트들을 하나의 그룹으로 묶을 건데, 그 그룹의 제목을 만드세요.
      `,
    },
  ],
  model: "gpt-4o-mini",
  temperature: 1.0,
});
```

첫 번째와 두 번째 방법 중 올바른 방법은 두 번째 방법이다. 왜냐하면 assistant message를 임의로 수정하면, 모델의 다음 응답이 매우 이상해지는 문제가 생기기 때문이다. 우리가 의도한, 요구한 내용을 응답하지 않는다.

내가 AI나 LLM의 원리에 대해 잘 알고 있는게 아니라서 이유는 잘 모르겠지만, 어쨋든 모델의 응답은 건드리지 않고 받은 그대로 사용하는게 좋다는 것은 확실하다.
