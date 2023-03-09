# vscode clangd添加include路径

编译settings.json

```json
"clangd.fallbackFlags": [
    	"-I<path-to-include>",
        "-I/usr/lib/jvm/java-17-openjdk/include",
        "-I/usr/lib/jvm/java-17-openjdk/include/linux"
    ]
```

