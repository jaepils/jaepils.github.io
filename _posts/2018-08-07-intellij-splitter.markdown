---
layout: post
title:  "Creating a Splitter"
date:   2018-08-07 15:40:56
categories: intelliJ
---
## Splitter
Splitter는 한 화면을 분리할때 많이 사용되는 컴포넌트입니다. 좌우 분리도 가능하고, 상하로 나눌 수 있습니다. Splitter는 반드시 2개의 서브 컴포넌트가 사용이 됩니다. 만약 Splitter를 만들고 하나의 컴포넌트만 추가하면 Splitter가 보이지 않습니다. 물론 3개의 컴포넌트는 추가할 방법이 없습니다. 이경우 재귀적으로 Splitter를 다시 사용을 해야합니다.

이번 작업 순서는 다음과 같습니다.
1. Splitter 생성
2. Test


### Splitter 생성
intelliJ에서 Splitter는 JBSplitter로 새로 만들었습니다. 사용법은 매우 간단합니다.

```java
public class CustomView {

    private JBSplitter mySplitter;

    public static CustomView getInstance(@NotNull Project project) {
        return project.getComponent(CustomView.class);
    }

    public void createContentPanel(@NotNull Project project, @NotNull ToolWindow toolWindow) {
        SampleForm panel1 = new SampleForm();
        SampleForm panel2 = new SampleForm();

        mySplitter = new JBSplitter();
        mySplitter.setFirstComponent(panel1.getMainPanel());
        mySplitter.setSecondComponent(panel2.getMainPanel());

        final Content content = ContentFactory.SERVICE.getInstance().createContent(mySplitter, "", false);
        content.setCloseable(true);

        toolWindow.getContentManager().addContent(content);
    }
}
```
JBSplitter를 만들고, firstComponent와 secondComponent를 추가를 하면 되는데, toolWindow의 content영역에서는 JBSplitter를 넘겨주면 됩니다. 이부분은 이전 탭과 조금 다른 부분입니다.

## 테스트
Splitter의 기본값은 vertical이 false즉 horizontal상태입니다.
<img width="956" alt="2" src="https://user-images.githubusercontent.com/23305428/43747076-b19e8bdc-9a22-11e8-8f59-ec630f59340e.png">

만약 Splitter를 생성시 vertical을 true로 하면 다음처럼 화면이 나옵니다.
<img width="957" alt="3" src="https://user-images.githubusercontent.com/23305428/43747105-e4daca60-9a22-11e8-8b80-a8e15a0d43d1.png">

가운데 부분을 마우스로 드래그하면 화면 크기가 바뀌게 됩니다.
