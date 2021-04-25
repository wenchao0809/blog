# 配置

### 默认配置

~~~js
export function getDefaultNuxtConfig (options = {}) {
  if (!options.env) {
    options.env = process.env
  }

  return {
    ..._app(),
    ..._common(),
    build: build(),
    messages: messages(),
    modes: modes(),
    render: render(),
    router: router(),
    server: server(options),
    cli: cli(),
    generate: generate()
  }
}
~~~