# 웹팩 빌드 Error: error:0308010C:digital envelope routines::unsupported

Github Actions에서 Workflow Runner를 `ubuntu-latest`로 설정하고서 잘 쓰고 있던 중, 갑자기 웹팩 빌드 단계에서 `Error: error:0308010C:digital envelope routines::unsupported` 에러가 발생했었다. 일단 [여기](../../../front-end/webpack/build-error-0308010c/)서 정리했듯이 해당 에러는 Node.js 버전이 16을 넘어갈 경우 발생하는 에러이다.

우분투에는 Node.js가 기본으로 내장되어있다. 이때 `ubuntu-latest`가 바라보는 우분투 버전은 (latest니까 당연히)계속 바뀔텐데, 바뀐 우분투 버전에 내장된 Node.js의 버전이 16을 초과하게 되어 영향이 온 것으로 보였다.

그래서 처음에는 내장 Node.js 버전이 16인 `ubuntu-18.04`를 Workflow Runner로 사용하도록 수정했고, 문제가 해결되어 잘 쓰고 있었다. 그런데 갑자기 [Ready 단계에서 Workflow Runner를 찾지 못하고 무한 대기하는 문제](../infinite-ready-job/)가 생겼었다.

하기야 Node.js 버전 이슈를 OS 버전을 바꿈으로써 해결하는 발상 자체가 지금 생각해보면 말도 안되는 발상이었다. 따라서 [actions/setup-node](https://github.com/actions/setup-node)를 사용하여 Node.js 버전을 16으로 고정하여 문제가 완전히 해결되었다.

```yaml
steps:
  - uses: actions/setup-node@v3
    with:
      node-version: 16
```
