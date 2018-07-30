---
layout: post
title:  "Creating an action"
date:   2018-07-30 15:40:56
categories: intelliJ
---
## plugin
이클립스와 마찬가지로 IntelliJ에 사용자가 플러그인을 만들수 있습니다. 플러그인은 시작은 우선 가장 간단한 버튼을 추가하는 것으로 시작을 보통합니다. 마치 Hello world처럼요.
우선 플러그인을 만드는건 java로 개발을 하면 되는데, 실제 만들고 나서 어플리케이션을 구동하게 되면 다시 IntelliJ가 새로 뜨게 됩니다.

상단 메뉴에 사용자가 클릭을 할 수 있는 액션 버튼을 만들어보겠습니다.

## 프로젝트 생성
일반 자바 프로젝트와 마찬가지로 플러그인도 프로젝트를 우선 생성해야합니다.

IntelliJ에서 새로운 프로젝트를 생성하면 다음과 같은 다이얼로그가 뜨게 됩니다.
<img width="897" alt="2018-07-30 6 41 10" src="https://user-images.githubusercontent.com/23305428/43390479-487f351a-9429-11e8-821f-a8848115e5d0.png">
여기에서 위의 그림처럼 IntelliJ Platform Plugin을 선택을 하면 Project SDK에 현재 IntelliJ의 버전이 나오고, 추가적인 라이브러리로 Groovy나 SQL Support를 선택할 수 있습니다.
우선 아무것도 선택을 하지 않고 다음을 누릅니다.

<img width="899" alt="2018-07-30 6 41 36" src="https://user-images.githubusercontent.com/23305428/43390485-4a198b50-9429-11e8-82ca-af5312d96cca.png">
프로젝트 명은 임의로 만드시면 되고, 저장될 위치도 임의로 지정을 하시면 됩니다.

<img width="414" alt="2018-07-30 6 52 42" src="https://user-images.githubusercontent.com/23305428/43390697-c8a1b89e-9429-11e8-834b-7304770c98e5.png">

생성된 프로젝트를 보면 META-INF밑에 plugin.xml이 있을 뿐입니다.
임의로 패키지를 만들고 다음 소스처럼 Action을 생성을 합니다.

```java
package sample.action;

import com.intellij.openapi.actionSystem.AnAction;
import com.intellij.openapi.actionSystem.AnActionEvent;
import com.intellij.openapi.actionSystem.PlatformDataKeys;
import com.intellij.openapi.project.Project;
import com.intellij.openapi.ui.Messages;

public class TextBoxes extends AnAction {
    // If you register the action from Java code, this constructor is used to set the menu item name
    // (optionally, you can specify the menu description and an icon to display next to the menu item).
    // You can omit this constructor when registering the action in the plugin.xml file.
    public TextBoxes() {
        // Set the menu item name.
        super("Text _Boxes");
        // Set the menu item name, description and icon.
        // super("Text _Boxes","Item description",IconLoader.getIcon("/Mypackage/icon.png"));
    }

    public void actionPerformed(AnActionEvent event) {
        Project project = event.getData(PlatformDataKeys.PROJECT);
        String txt= Messages.showInputDialog(project, "What is your name?", "Input your name", Messages.getQuestionIcon());
        Messages.showMessageDialog(project, "Hello, " + txt + "!\n I am glad to see you.", "Information", Messages.getInformationIcon());
    }
}
```
intelliJ에서 제공하는 openapi에서 AnAction이라는 상위 클래스를 제공을 합니다. actionPerformed는 실제 클릭 이벤트가 발생이 되었을때 호출이 됩니다.

actionPerformed의 코드를 보면 첫번째라인에서 Project를 구하는 것을 알 수 있습니다. 현재 구동된 워크스페이스에서의 프로젝트를 의미합니다. 구한 project를 파라미터로 showInputDialog를 하게 되면 팝업이 뜨게 됩니다.
이름을 물어보고 getQuestionIcon은 당연히 ?가 나오는 다이얼로그라는 의미입니다.
이름을 구하면 다음에는 다시 메세지 다이얼로그를 띄우는데, 이때도 파라미터는 project입니다.

이렇게 Action 클래스를 생성을 하면 IntelliJ에서 이 클래스의 존재를 알게 해야합니다.
방법은 초기에 자동 생성된 xml에 action을 등록을 하면 됩니다.

```java
<actions>
  <!-- Add your actions here -->
  <group id="MyPlugin.SampleMenu" text="_Sample Menu" description="Sample menu">
    <add-to-group group-id="MainMenu" anchor="last"  />
    <action id="Myplugin.Textboxes" class="sample.action.TextBoxes" text="Text _Boxes" description="A test menu item" />
  </group>
</actions>
```
그룹으로 Sample menu를 만들고, MainMenu의 가장 끝 즉 IntelliJ에서 상단 메뉴의 가장 끝에 오게 합니다.
그 하단으로 지금 만든 action을 등록을 합니다.

실행하면 다음과 같은 팝업이 뜨게 됩니다.

<img width="499" alt="2018-07-30 7 00 56" src="https://user-images.githubusercontent.com/23305428/43391168-eadd52a0-942a-11e8-872b-0a4eaed4df82.png">

상단 메뉴 Help 다음에 Sample Menu가 나오게 됩니다. 그 하단에는 Text Boxes가 나오고 클릭을 하면 이름을 묻는 팝업이 뜨게 됩니다.

## 참조
* https://www.jetbrains.org/intellij/sdk/docs/basics/getting_started/creating_an_action.html
