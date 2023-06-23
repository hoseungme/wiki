# Waiting for a runner to pick up this job 무한 대기

Github Actions에서 가장 처음 거치는 단계는 Workflow가 실행될 Runner를 세팅하는 것인데, 이때 그 단계에서 넘어가지 못하고 무한 대기하는 이슈가 있었다.

Runner를 설정하는 옵션은 [jobs.job.runs-on](https://docs.github.com/ko/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idruns-on)인데, 여러 StackOverflow, Github Issue 등을 찾아보니 `runs-on`에 넘긴 값이 잘못되어 그 값에 대응하는 Workflow Runner를 찾을 수 없는 경우 다음 단계로 넘어가지 못한다고 한다.

하지만 내가 사용하던 값인 `ubuntu-18.04`에는 오타도 없었고, 기존에 멀쩡히 잘 돌아갔던 값인데, 오랜만에 커밋을 하니 갑자기 무한루프에 걸린 것이었다.

아무래도 `ubuntu-18.04` Runner에 대한 Github Actions의 지원이 끊어지거나 한게 분명해 보였고, [그 예상은 적중](https://github.com/actions/runner-images/issues/6002)했다.

이전부터 더이상 지원되지 않으며 곧 완전히 삭제된다는 언급이 되었었는데, 그 삭제한다는 날이 2023년 4월 1일이었던 것.

따라서 그냥 최신 우분투 버전을 따라가도록 `ubuntu-latest`로 바꿔주니 무한 대기 문제가 해결되었다.

다만, 사실 과거의 나는 원래 `ubuntu-latest`를 사용하고 있었으나, 웹팩 문제 때문에 `ubuntu-18.04`로 버전을 고정해서 사용하게 된 것이었는데, 그것은 [여기](../webpack-error-0308010c/ko.md)서 설명한다.
