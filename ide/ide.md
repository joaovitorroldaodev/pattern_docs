# Configurações de arquivos padrões para IDEs e pastas de projetos

## .prettierrc

```json
{
  "useTabs": false,
  "semi": false,
  "arrowParens": "always",
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "es5",
  "bracketSpacing": true,
  "printWidth": 100,
  "endOfLine": "lf",
  "bracketSameLine": false,
  "proseWrap": "preserve"
}
```

## .editorconfig

```
[*]
indent_style = space
indent_size = 2
end_of_line = lf
charset = utf-8
trim_trailing_whitespace = true
insert_final_newline = true
```

## .gitignore

```
# Logs (opcional)
logs
*.log
npm-debug.log*
yarn-debug.log*
yarn-error.log*
pnpm-debug.log*
lerna-debug.log*

# build (opcional)
node_modules
dist
dist-ssr
*.local
build

# Editor directories and files (required)
.vscode/*
!.vscode/extensions.json
.idea
.DS_Store
*.suo
*.ntvs*
*.njsproj
*.sln
*.sw?
```
