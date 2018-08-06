---
layout: post
title:  "Creating a tab"
date:   2018-08-07 15:40:56
categories: intelliJ
---
## Tab
Tab은 하나의 화면에서 여러 정보를 보여줄때 많이 쓰입니다. Swing에서 제공하는 JTabbedPane이 아닌 intellij에서 만든 JBTabs 사용법을 알아보도록 하겠습니다.
이번 예제부터는 이전에 사용하던 toolWindow를 재사용하도록 하겠습니다.

이번 작업 순서는 다음과 같습니다.
1. Tab 생성
2. Test


### Tab 생성
JBTabs은 탭을 관리하는 JBTabs와 탭안에서 실제 화면을 구성하는 TabInfo가 사용이 됩니다.
즉 JBTabs은 여러 TabInfo를 가지고 탭을 구성을 합니다. TabInfo는 서로 독립적인 공간으로 서로 다은 UI를 만들 수 있습니다.

```java
public class CustomView {

    private JBTabs myTabsPane;

    public static CustomView getInstance(@NotNull Project project) {
        return project.getComponent(CustomView.class);
    }

    public void createContentPanel(@NotNull Project project, @NotNull ToolWindow toolWindow) {
        SampleForm panel = new SampleForm();

        TabInfo tab1 = new TabInfo(panel.getMainPanel());
        tab1.setText("tab1");

        myTabsPane = new JBTabsImpl(project);
        myTabsPane.addTab(tab1);

        final Content content = ContentFactory.SERVICE.getInstance().createContent(myTabsPane.getComponent(), "", false);
        content.setCloseable(true);

        toolWindow.getContentManager().addContent(content);
    }
}
```
우선, TabInfo는 탭안의 화면을 가지고 있습니다. 위의 코드에서는 생성자로 넘겨주고 있습니다. 탭은 아이콘과 텍스트가 사용이 되는데, 텍스트만 입력을 하였습니다.

이렇게 만들어진 TabInfo는 JBTabs에 addTab으로 추가를 해주기만 하면 됩니다.
ToolWindow의 Component로는 JBTabs의 getComponent를 넘겨주기만 하면 됩니다.

JBTabs은 이외에도 마우스 이벤트, Drag & Drop기능도 제공하지만, 다음에 설명하도록 하겠습니다.

## 테스트

<img width="956" alt="1" src="https://user-images.githubusercontent.com/23305428/43745820-c5e817f4-9a1b-11e8-8d73-4c6f88b7872c.png">

이전에 만든 toolWindow에 상단에 tab1 하나 생겼습니다.
Form에서 만든 버튼도 화면에 나오게 됩니다.
