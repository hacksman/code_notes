以修改创建文件的行为为例，默认 vscode 按快捷键 `⌘ + n` 创建文件是以 `workbench.action.files.newUntitledFile` 模式创建文件的，非常不方便使用。

![[Pasted image 20210926173327.png]]

现在需要将它改成 `explorer.newFile` 的方式。`vscode` 中的很多功能都是通过修改文件的配置项完成的，我们通过快捷键 `⌘ + ⇧ + p` 快速唤起查找功能，然后找到修改快捷键绑定的配置项 `Preferences: Open Keyboard Shortcuts (JSON)`。

![[Pasted image 20210926173657.png]]

然后修改对应的对应的绑定即可

![[Pasted image 20210926173754.png]]

改完之后就可以愉快地按照我们之前的习惯创建文件夹啦！

![[Pasted image 20210926173847.png]]

## 参考资料
[VS Code - Add a new file under the selected working directory](https://stackoverflow.com/questions/39599514/vs-code-add-a-new-file-under-the-selected-working-directory)