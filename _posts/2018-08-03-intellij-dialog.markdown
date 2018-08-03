---
layout: post
title:  "Creating a dialog"
date:   2018-08-03 15:40:56
categories: intelliJ
---
## Dialog
IntelliJ에서 다이얼로그를 만들어보도록 하겠습니다. 다이얼로그는 특정 설정을 하거나 사용자에게 정보를 보여주는 용도로 모달로 주로 사용이 됩니다.

이번 작업 순서는 다음과 같습니다.
1. Dialog 생성
2. Action에서 dialog 초기화
3. Test


### dialog 생성
Form과 마찬가지로 dialog도 intelliJ에서 메뉴를 제공하고 있습니다.

<img width="594" alt="1" src="https://user-images.githubusercontent.com/23305428/43620617-a230f03e-970e-11e8-9c99-d40e161b3ba7.png">

New/Dialog를 선택하면 다음과 같은 팝업이 뜨게 됩니다.

<img width="388" alt="2" src="https://user-images.githubusercontent.com/23305428/43620623-a5032836-970e-11e8-9bcb-11e2cf335283.png">

Dialog클래스명을 입력하고 하단 3개 옵션중 필요한 것을 선택하면 됩니다.
- Generate main : 생성된 Dialog를 plugin형태가 아닌 별도 어플리케이션으로 띄울건지 선택
- Generate OK : OK 버튼 여부
- Generate Cancel : Cancel 버튼 여부

일단 전부 체크를 하고 넘어갑니다.

<img width="341" alt="3" src="https://user-images.githubusercontent.com/23305428/43620624-a70d3cde-970e-11e8-8e01-8c44d887dd4d.png">

만들어진 파일은 2개입니다. SampleDialog.form과 SampleDialog.java파일입니다.
Form에서 봤듯이 화면 구성을 담당하는 xml파일인 form과 이벤트를 처리하는 java파일이 생성이 됩니다.

Form의 경우 빈화면이였지만, Dialog는 다음 그림처럼 하단에 OK, Cancel버튼이 있습니다.

<img width="482" alt="4" src="https://user-images.githubusercontent.com/23305428/43620777-5150b342-970f-11e8-8189-82a91b4375d6.png">

여기에 간단하게 버튼을 하나 추가를 합니다.

```java
public class SampleDialog extends JDialog {
    private JPanel contentPane;
    private JButton buttonOK;
    private JButton buttonCancel;
    private JButton button1;

    public SampleDialog() {
        setContentPane(contentPane);
        setModal(true);
        getRootPane().setDefaultButton(buttonOK);

        buttonOK.addActionListener(new ActionListener() {
            public void actionPerformed(ActionEvent e) {
                onOK();
            }
        });

        buttonCancel.addActionListener(new ActionListener() {
            public void actionPerformed(ActionEvent e) {
                onCancel();
            }
        });

        // call onCancel() when cross is clicked
        setDefaultCloseOperation(DO_NOTHING_ON_CLOSE);
        addWindowListener(new WindowAdapter() {
            public void windowClosing(WindowEvent e) {
                onCancel();
            }
        });

        // call onCancel() on ESCAPE
        contentPane.registerKeyboardAction(new ActionListener() {
            public void actionPerformed(ActionEvent e) {
                onCancel();
            }
        }, KeyStroke.getKeyStroke(KeyEvent.VK_ESCAPE, 0), JComponent.WHEN_ANCESTOR_OF_FOCUSED_COMPONENT);
    }

    private void onOK() {
        // add your code here
        dispose();
    }

    private void onCancel() {
        // add your code here if necessary
        dispose();
    }

    public static void main(String[] args) {
        SampleDialog dialog = new SampleDialog();
        dialog.pack();
        dialog.setVisible(true);
        System.exit(0);
    }
}
```
이것만으로도 Dialog.java파일은 위의 코드처럼 조금 복잡하게 생성이 됩니다.
우선 버튼이 있기 때문에 버튼 클릭 이벤트일때 dialog를 닫는 dispose가 실행이 되도록 코드가 생성이 됩니다.
하단의 main은 dialog 생성 위자드에서 Generate Main을 선택하면 추가되는 코드입니다.
여기서 바로 실행해보면 dialog ui를 볼 수 있습니다.

Swing에서는 dialog 생성시 pack을 호출한 후 setVisible을 호출해야합니다.
pack을 호출을 안하게 되면 dialog가 컨텐츠 영역 전체 크기를 계산을 못하기 때문에 조그마한 크기로 뜨게 됩니다.

### Action에서 dialog 초기화
이전처럼 처음 생성했던 action에 dialog만 띄우는 기능을 추가하겠습니다.

```java
public class CustomDialogAction extends AnAction {

    public CustomDialogAction() {
        super("CustomToolWindow");
    }

    public void actionPerformed(AnActionEvent event) {
        Project project = event.getData(PlatformDataKeys.PROJECT);

        final SampleDialog dlg = new SampleDialog();
        dlg.pack();

        dlg.setVisible(true);
    }
}
```
이전에 설명했듯이, pack을 호출하고 setVisible을 실행합니다.

## 테스트

<img width="539" alt="5" src="https://user-images.githubusercontent.com/23305428/43621010-4f509250-9710-11e8-9b09-8dcef02e3e93.png">

action을 클릭하면 그림처럼 조그마한 dialog가 나오게 됩니다. 이전처럼 버튼 클릭시 아무런 반응은 없습니다.

## 주의
여러 플러그인을 개발하다보면 예전 플러그인만 실행이 되면 아래 디렉토리에서 플러그인 폴더들을 지워주시면 됩니다.
./Library/Caches/IntelliJIdea2018.2/plugins-sandbox/plugins
