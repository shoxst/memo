# VBAの基礎と継承の実装

## 基礎
### モジュール
- 標準モジュール
  - シートを指定しないセルはアクティブシートのセルとみなされる。
  - 自身のインスタンスを表す`Me`は使用できない。
- シートモジュール
  - シートを指定しないセルはそのシートのセルとみなされる。

定数、固定長文字列、配列、構造体をPublicで宣言できるのは標準モジュールのみ。他のクラス(シートモジュールなど)では宣言できない。

### 変数
スコープは以下の通り。
- `Public`
  - すべてのファイルから参照できる
- `Private`
  - 宣言しているファイル内から参照できる
- `Dim`
  - 宣言した関数の中でのみ参照できる

### 関数
スコープは、`Public`と`Private`がある。指定しない場合は`Public`。

`ByVal`(値渡し)と`ByRef`(参照渡し)がある。省略すると参照渡しになる。基本は`ByVal`を利用する。

### ショートカット
- VBEを開く
  - Alt + F11
- マクロを実行
  - Alt + F8

## 継承の実装
VBAでは継承がサポートされていないので、interfaceを使ってそれっぽいものを実装する。

引数ありコンストラクタはないので、とりあえず関数実装で対処。

IInheritable.cls
```vb
Public Property Get Super() As Object
End Property
```

IPerson.cls
```vb
Public Function ToString() As String
End Function
```

Person.cls
```vb
Public This As IPerson
Public Name As String
Public Age As Integer

Public Sub Init(ByVal pName As String, ByVal pAge As Integer)
    Name = pName
    Age = pAge
End Sub

Public Function ToString() As String
    ToString = This.ToString
End Function

Public Function ToStringBase() As String
    ToStringBase = "名前：" & Name & " 年齢：" & Age & "歳"
End Function

```

Student.cls
```vb
Implements IPerson
Implements IInheritable

Public Super As New Person
Public Id As Integer

Public Sub Init(ByVal pName As String, ByVal pAge As Integer, ByVal pId As Integer)
    Super.Init pName, pAge
    Id = pId
End Sub

Private Function IPerson_ToString() As String
    IPerson_ToString = Super.ToStringBase & " ID：" & Id
End Function
 
Private Property Get IInheritable_Super() As Object
    Set IInheritable_Super = Super
End Property
```

Module
```vb
Public Sub Main()
    Dim aPerson As Person, aStudent As New Student
    aStudent.Init "AAA", 15, 1
    Set aPerson = CreateInstance(aStudent)
   
    ' AAA
    Debug.Print aStudent.Super.Name

    ' 名前：AAA 年齢：15歳 ID：1
    Debug.Print aPerson.ToString
    
    ChangeStudentIdByAnotherFunction aPerson
    
    ' 名前：AAA 年齢：15歳 ID：2
    Debug.Print aPerson.ToString

End Sub

Private Function CreateInstance(ByVal SubClass As IInheritable) As Person
    Set CreateInstance = SubClass.Super
    Set CreateInstance.This = SubClass
End Function

Private Sub ChangeStudentIdByAnotherFunction(ByRef aPerson As Person)
    Dim aStudent As Student
    Set aStudent = aPerson.This
    aStudent.Id = 2
End Sub
```