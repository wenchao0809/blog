child_process 模块提供了衍生子进程（以一种与 popen(3) 类似但不相同的方式）的能力。 此功能主要由 child_process.spawn() 函数提供：

### `child_process.spawn()`


`child_process.spawn()` 方法会异步地衍生子进程，且不阻塞 `Node.js` 事件循环。 `child_process.spawnSync()` 函数则以同步的方式提供了等效的功能，但会阻塞事件循环直到衍生的进程退出或被终止。

* `child_process.exec()`: 衍生 `shell` 并且在 `shell` 中运行命令，当完成时则将 `stdout` 和 `stderr` 传给回调函数。
*` child_process.execFile()`: 类似于 `child_process.exec()`，但是默认情况下它会直接衍生命令而不先衍生 `shell`。
* `child_process.fork()`: 衍生新的 `Node.js` 进程，并调用指定的模块，该模块已建立了 `IPC` 通信通道，可以在父进程与子进程之间发送消息。
* `child_process.execSync()`: `child_process.exec()` 的同步版本，会阻塞 `Node.js` 事件循环。
* `child_process.execFileSync()`: `child_process.execFile()` 的同步版本，会阻塞 `Node.js` 事件循环。

对于某些用例，例如自动化的 `shell` 脚本，同步的方法可能更方便。 但是在大多数情况下，同步的方法会对性能产生重大的影响，因为会暂停事件循环直到衍生的进程完成。

`child.spawn`在`Windows`平台使用时可能会有一些问题可以使用`npm` 包`cross-spawn`