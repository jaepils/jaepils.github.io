---
layout: post
title:  "Creating a tool window #1"
date:   2018-08-01 15:40:56
categories: intelliJ
---
## ToolWindow
Tool windows는 정보를 보여주는 IDE의 child windows입니다. 이클립스에서는 view라고 불렸습니다. tool window는 자신만을 위한 툴바를 가질수 있고 사용자가 열고 닫을 수 특징이 있습니다.

실제 만들 tool window는 다음과 같습니다.

<img width="600" alt="1" src="https://user-images.githubusercontent.com/23305428/43493267-0771d4ac-9568-11e8-93c3-ec7a63bb2925.png">

하단에 CustomToolWindow가 이번에 만들어 볼 화면입니다. 색이 회색인건 아무런 컨텐츠를 생성하지 않아서입니다.
우선 Tool window의 기본 속성을 알아보겠습니다.
* **id (required)** : tool window의 유니크한 아이디입니다.
* **anchor (required)** : 보여줄 위치입니다. (left, right, bottom)
* secondary : Tool window가 기본 또는 보조 그룹에 표시되는지 여부 지정합니다.
* icon : 13 * 13 아이콘
* **factoryClass (required)** : tool window를 생성할때 필요한 클래스입니다. ToolWindowFactory를 implements해야합니다.

ToolWindow 클래스는 인터페이스입니다. 따라서 이 인터페이스를 구현한 클래스를 만들어야한다고 생각할 수 있는데, 실제로는 ToolWindow 의 구현체는 이미 있고 안의 컨텐츠만 구현을 하면 됩니다.
작업 순서는 다음과 같습니다.
1. CustomView 생성 : 실제 tool window의 컨텐츠를 보여줄 역할을 수행합니다.
2. CustomToolWindowFactory : CustomView를 생성합니다.
3. plugin.xml : 사용자 tool window를 등록을 합니다.
4. action : 클릭시 tool window를 오픈합니다.


### CustomView
CustomView는 아무런 인터페이스를 implements할 필요가 없습니다.

```java
public class CustomView {

    public static CustomView getInstance(@NotNull Project project) {
        return project.getComponent(CustomView.class);
    }
}
```
실제 코드는 project클래스를 통해서 CustomView의 인스턴스를 반환하는 클래스 뿐입니다.
인스턴스를 직접 생성하지 않는 이유는 IntelliJ에서 생성된 ToolWindow를 관리를 따로 하고 있어서 사용자가 window를 닫고 다시 열게 될때 새로 생성하지 않고 기존 tool window를 다시 반환을 합니다.

### CustomToolWindowFactory
위에서 언급한대로 CustomToolWindowFactory는 반드시 ToolWindowFactory를 implements해야합니다.

```java
public class CustomToolWindowFactory implements ToolWindowFactory {

    @Override
    public void createToolWindowContent(@NotNull Project project, @NotNull ToolWindow toolWindow) {
        CustomView customView = CustomView.getInstance(project);
    }
}
```
ToolWindowFactory에서는 createToolWindowContent를 구현하도록 하는데, 지금 필요한것은 CustomView의 인스턴스를 생성하는 역할을 합니다. 코드를 보면 생성된 CustomView는 리턴이 필요하지 않습니다. 이 한줄에 의해서 ToolWindow와 CustomView는 서로 연결이 되었습니다.

### plugin.xml
plugin.xml은 intelliJ가 로딩이 되면서 읽어 들이는 파일입니다.
지금 만든 toolWindow를 등록을 해주어야 합니다.
```xml
<extensions defaultExtensionNs="com.intellij">
    <!-- Add your extensions here -->
    <toolWindow id="CustomToolWindow" anchor="bottom"
                factoryClass="sample.ui.CustomToolWindowFactory"/>
</extensions>

<actions>
    <!-- Add your actions here -->
    <group id="MyPlugin.SampleMenu" text="_Sample Menu" description="Sample menu">
      <add-to-group group-id="MainMenu" anchor="last"  />
      <action id="Myplugin.CustomToolWindow" class="sample.action.CustomToolWindowAction" text="Open Custom tool" />
    </group>
</actions>

<project-components>
  <component>
      <interface-class>sample.ui.view.CustomView</interface-class>
      <implementation-class>sample.ui.view.CustomView</implementation-class>
  </component>
</project-components>
```
id는 CustomToolWindow에 위치는 하단, 생성은 CustomToolWindowFactory를 통해서 하도록 했습니다.

### action
action은 이전에 만든 것을 재활용하겠습니다. 단지 클릭 이벤트만 필요합니다.
```java
public class CustomToolWindowAction extends AnAction {

    public CustomToolWindowAction() {
        super("CustomToolWindow");
    }

    public void actionPerformed(AnActionEvent event) {
        Project project = event.getData(PlatformDataKeys.PROJECT);

        ToolWindow window = ToolWindowManager.getInstance(project).getToolWindow("CustomToolWindow");
        if (window != null && window.isAvailable()) {
            window.activate(null);
        }
    }
}
```
클릭시 ToolWindow를 project에서 얻는 요청을 하면 intelliJ에서 CustomToolWindowFactory의 createToolWindowContent를 실행합니다. 생성이 된 경우에 activate를 호출하면 화면에 보여지게 됩니다.

## 테스트
상단에 Sample menu 하단에 Open Custom tool 클릭시 처음에 봤던 toolWindow가 실행이 됩니다.

## 참조
* https://www.jetbrains.org/intellij/sdk/docs/user_interface_components/tool_windows.html
