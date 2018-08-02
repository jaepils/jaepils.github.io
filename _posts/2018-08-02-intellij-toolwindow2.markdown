---
layout: post
title:  "Creating an tool window #2"
date:   2018-08-01 15:40:56
categories: intelliJ
---
## Form
IntelliJ의 기본 UI component는 SwingUI를 사용하고 있습니다. java에서 배운대로 UI를 구성도 가능하지만, 손쉽게 UI를 만들수 있는 Form Editor를 제공하고 있고, UI에 관한 속성은 xml에 정의되고, 실제 이벤트 처리만 java에서 처리합니다.

이번 작업 순서는 다음과 같습니다.
1. form 생성
2. CustomView에 UI 초기화 하는 함수 생성
3. CustomToolWindowFactory에서 UI 초기화 호출
4. Test


### form 생성
이전에 만든 프로젝트에서 New를 하게 되면 GUI Form이라는 메뉴가 나옵니다.
<img width="594" alt="1" src="https://user-images.githubusercontent.com/23305428/43556414-b0c33be0-963a-11e8-9dc1-8a9f9e8c90f6.png">

GUI Form을 클릭하면 아래와 같은 다이얼로그가 나옵니다.

<img width="418" alt="2" src="https://user-images.githubusercontent.com/23305428/43556455-cf80b99a-963a-11e8-83ce-cc32af12c118.png">

Form name을 적고, 화면에 배치될 layout을 명시합니다. 여기서는 기본값인 GridLayoutManager를 그대로 둡니다.

<img width="379" alt="3" src="https://user-images.githubusercontent.com/23305428/43556475-d25a7d40-963a-11e8-8c22-94b327c7f80c.png">

OK를 누르면 파일 2개가 만들어집니다.
- SampleForm.form : UI 담당
- SampleForm.java : 이벤트 담당

<img width="1038" alt="4" src="https://user-images.githubusercontent.com/23305428/43556477-d45daa7c-963a-11e8-847d-b12b2f57e19b.png">

우선 form을 클릭하면 위와 같이 Swing GUI designer가 나옵니다. 쉽게 drag만으로 화면 구성이 가능합니다. 여기서 상단 Jpanel에 field name을 mainPanel로 정의를 하고 간단하게 버튼을 추가를 합니다.
저장을 하게 되면 SampleForm.java이 수정이 됩니다.


```java
public class SampleForm {
    private JButton button1;
    private JPanel mainPanel;

    public JPanel getMainPanel() {
        return mainPanel;
    }
}
```
실제 코드에서는 layout에 관한 코드가 없습니다. 이부분이 저는 개인적으로 정말 좋은거 같습니다. UI를 하게 되면 가장 복잡한 부분인데, 눈에 보이지 않는다는 것만으로도 행복해지는거 같습니다.

form의 layout은 xml에 다음처럼 수정이 됩니다
```xml
<?xml version="1.0" encoding="UTF-8"?>
<form xmlns="http://www.intellij.com/uidesigner/form/" version="1" bind-to-class="sample.ui.panel.SampleForm">
  <grid id="27dc6" binding="mainPanel" layout-manager="GridLayoutManager" row-count="2" column-count="1" same-size-horizontally="false" same-size-vertically="false" hgap="-1" vgap="-1">
    <margin top="0" left="0" bottom="0" right="0"/>
    <constraints>
      <xy x="20" y="20" width="500" height="400"/>
    </constraints>
    <properties/>
    <border type="none"/>
    <children>
      <component id="15be4" class="javax.swing.JButton" binding="button1" default-binding="true">
        <constraints>
          <grid row="0" column="0" row-span="1" col-span="1" vsize-policy="0" hsize-policy="3" anchor="0" fill="1" indent="0" use-parent-layout="false"/>
        </constraints>
        <properties>
          <text value="Button"/>
        </properties>
      </component>
      <vspacer id="852c8">
        <constraints>
          <grid row="1" column="0" row-span="1" col-span="1" vsize-policy="6" hsize-policy="1" anchor="0" fill="2" indent="0" use-parent-layout="false"/>
        </constraints>
      </vspacer>
    </children>
  </grid>
</form>
```
이 코드가 예전에는 java에 있었습니다.

### CustomView에 UI 초기화 하는 함수 생성
저번에 만든 CustomView 클래스에 createContentPanel이라는 메소드를 생성을 합니다.

```java
public class CustomView {

    public static CustomView getInstance(@NotNull Project project) {
        return project.getComponent(CustomView.class);
    }

    public void createContentPanel(@NotNull ToolWindow toolWindow) {
        SampleForm panel = new SampleForm();

        final Content content = ContentFactory.SERVICE.getInstance().createContent(panel.getMainPanel(), "", false);
        content.setCloseable(true);

        toolWindow.getContentManager().addContent(content);
    }
}
```
toolWindow에게 컨텐츠를 추가를 해줘야 하는데 Content가 실제 화면구성인 mainPanel을 넘겨줘서 생성을 하면 됩니다. 즉 SampleForm은 실제 화면에 관련된 클래스가 아니기 때문에 SampleForm의 mainPanel을 넘겨줘야 화면이 그려지게 됩니다.

### CustomToolWindowFactory에서 UI 초기화 호출
기존 Factory에서는 customView만 생성하는게 아니라 UI까지 초기화를 해줍니다.

```java
public class CustomToolWindowFactory implements ToolWindowFactory {

    @Override
    public void createToolWindowContent(@NotNull Project project, @NotNull ToolWindow toolWindow) {
        CustomView customView = CustomView.getInstance(project);
        customView.createContentPanel(toolWindow);
    }
}
```
customView.createContentPanel을 호출하면 toolWindow의 컨텐츠 영역에 panel이 등록이 됩니다.


## 테스트
이제 다시 IntelliJ을 시작을 하게 되면 다음과 같은 화면이 나오게 됩니다.

<img width="963" alt="5" src="https://user-images.githubusercontent.com/23305428/43557715-ff1b1c44-9640-11e8-8733-f5e8630fe2b1.png">

물론 아직 이벤트를 붙이지 않았기 때문에 버튼을 클릭해도 아무런 반응은 없습니다.


## 참조
* https://www.jetbrains.com/help/idea/components-of-the-gui-designer.html
