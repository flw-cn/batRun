## batRun 设计概要

### 定义、定位与背景

batRun 用来在一批设备上完成给定的作业内容。

### 术语定义

整个系统涉及到的概念如下：

* BatchJob
> 批处理作业
> 一个批处理作业由多台主机以及位于其上的作业描述构成。

* HostInfo
> 主机信息
> 一个主机信息包括主机名、端口号，以及一个可用的账号信息。
> 主机登录成功之后，会保持一个 SSH 连接句柄。

* AccountInfo
> 账号信息
> 一个账号信息包括了用户名、口令等信息。

* Job
> 作业描述
> 一个作业就是指一系列的任务。

* Task
> 任务描述
> 凡是能够在设备上执行并产生一定输出的动作，都可以称为任务。

* TaskResult
> 任务处理结果
> 任务处理结果包含了一个文本型的输出结果，以及一个错误码。
> 特别的，如果是指令类任务的话，batRun 也可以将标准错误输出和标准输出的内容予以区分。

* CommandTask
> 指令类任务
> 指令类任务是指发送一条 Shell 命令，并捕获其输出的任务。
> 指令类任务的执行结果就是这条命令的屏幕输出内容。

* UploadTask
> 上传文件任务
> 向主机上传一个文件的任务。

* DownloadTask
> 下载文件任务
> 从主机上下载一个文件的任务。

* 用户自定义任务
> 用户也可以自己定义任务，从而扩展 batRun 的处理能力。
> 用户自定义的任务需要提供一个 Exec 方法来执行，并且能够输出 TaskResult 类型的结果。
> batRun 为用户自定义任务提供了 SSH 通道，用户可以通过 Exec 方法的 ssh.Session 参数来访问远程主机。

### 主要数据结构

```Go
type HostInfo struct {
	Host   string     // 主机名
	Port   string     // 端口号
	Handle ssh.Client // SSH 句柄
}

type AccountInfo struct {
	User string // 用户名
	Pass string // 口令
}

type Task interface {
	// 任务的 ID
	ID() string

	// 执行任务
	Exec(session ssh.Session) TaskResult
}

type TaskResult struct {
	// 任务在执行时产生的标准输出内容
	stdout string

	// 任务在执行时产生的标准错误输出内容
	stderr string

	// 如果任务的输出能够分为标准输出和标准错误输出的话，
	// 这里就是两者的混合，就像是你在显示器上看到的一样；
	// 如果不能够区分的话，那么 output 就是任务的输出
	output string

	// 任务执行后的结果
	err error

	// 任务经过裁决后的结论
	verdict string
}

type TaskMetaData struct {
	id string
}

type TaskJudge struct {
}

type CmdTask struct {
	TaskMetaData
	TaskJudge
	cmd string
}

type Job []Task

type JobResult struct {
}
```
