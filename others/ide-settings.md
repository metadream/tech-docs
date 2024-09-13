# IDE Settings

## Sublime Text

```
// settings.json
{
    "font_face": "JetBrains Mono",
    "font_size": 11,
    "line_padding_top": 2,
    "line_padding_bottom": 2,
    "trim_trailing_white_space_on_save": true
}

// keymap.json
[
	{ "keys": ["ctrl+shift+x"], "command": "upper_case" },
	{ "keys": ["ctrl+shift+l"], "command": "lower_case" },
	{ "keys": ["ctrl+d"], "command": "run_macro_file", "args": {"file": "res://Packages/Default/Delete Line.sublime-macro"} }
]
```

## VS Code

```
// settings.json
{
    "workbench.colorTheme": "Default Light Modern",
    "editor.fontFamily": "'JetBrains Mono'",
    "editor.minimap.enabled": false,
    "workbench.startupEditor": "none",
    "deno.enable": true,
    "files.trimTrailingWhitespace": true,
    "[typescript]": {
        "editor.defaultFormatter": "vscode.typescript-language-features"
    },
    "[markdown]": {
        "editor.defaultFormatter": "denoland.vscode-deno"
    },
    "editor.formatOnSave": true,
    "editor.detectIndentation": false
}

// keybindings.json
// Place your key bindings in this file to override the defaults
[
    {
        "key": "ctrl+d",
        "command": "-editor.action.addSelectionToNextFindMatch",
        "when": "editorFocus"
    },
    {
        "key": "ctrl+d",
        "command": "editor.action.deleteLines",
        "when": "textInputFocus && !editorReadonly"
    },
    {
        "key": "ctrl+shift+k",
        "command": "-editor.action.deleteLines",
        "when": "textInputFocus && !editorReadonly"
    },
    {
        "key": "ctrl+shift+x",
        "command": "-workbench.view.extensions",
        "when": "viewContainer.workbench.view.extensions.enabled"
    },
    {
        "key": "ctrl+shift+x",
        "command": "editor.action.transformToUppercase"
    },
    {
        "key": "ctrl+shift+i",
        "command": "-chat.action.askQuickQuestion",
        "when": "config.chat.experimental.quickQuestion.enable && hasChatProvider"
    },
    {
        "key": "ctrl+shift+l",
        "command": "editor.action.transformToLowercase"
    }
]
```

## Eclipse

[下载](https://cdn.unpkg.net/~metadream/backend@1.0.1/settings/)
格式化配置文件，分别导入 Eclipse 中 Java/Javascript（后者需下载 Web Developer
Tools 插件） 自定义 Formatter。
