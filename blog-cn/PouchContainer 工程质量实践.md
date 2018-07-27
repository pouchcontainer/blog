# 0. 前言

随着 [PouchContainer](https://github.com/alibaba/pouch) 功能不断地迭代和完善，项目也逐渐庞大起来，这吸引了不少外部开发者来参与项目的开发。由于每位贡献者编码习惯都不尽相同，代码审阅者的责任不仅仅是关注逻辑正确性和性能问题，还应该关注代码风格，因为统一的代码规范是保证项目代码可维护的前提。除了统一项目代码风格之外，测试用例的覆盖率和稳定性也是项目关注的重点。简单设想下，在缺少回归测试用例的项目，如何保证每次代码更新都不会影响到现有功能？

本文会分享 PouchContainer 在代码风格规范和 golang 单元测试用例方面的实践。

# 1. 统一的编码风格规范

PouchContainer 是由 golang 语言构建的项目，项目里会使用 shell script 来完成一些自动化操作，比如编译和打包操作。除了 golang 和 shell script 以外，PouchContainer 还包含了大量 Markdown 风格的文档，它是使用者认识和了解 PouchContainer 的入口，它的规范排版和正确拼写也是项目的关注对象。接下来的内容将会介绍 PouchContainer 在编码风格规范上使用的工具和使用场景。

## 1.1 Golinter - 统一代码格式

golang 的语法设计简单，加上社区一开始都有完备的 [CodeReview](https://github.com/golang/go/wiki/CodeReviewComments) 指导，让绝大部分的 golang 项目都有相同的代码风格，很少陷入到无谓的 __宗教__ 之争。在社区的基础上，PouchContainer 还定义了一些特定的规则来约定开发者，目的是为了保证代码的可读性，具体内容可阅读[这里](https://github.com/alibaba/pouch/blob/master/docs/contributions/code_styles.md#additional-style-rules)。

但光靠书面协议去做规范，这是很难保证项目代码风格保持一致。因此 golang 和其他语言一样，其官方提供了基础的工具链，比如 [golint](https://github.com/golang/lint),  [gofmt](https://golang.org/cmd/gofmt)，[goimports](https://github.com/golang/tools/blob/master/cmd/goimports/doc.go) 以及 [go vet](https://golang.org/cmd/vet) 等等，这些工具可在编译前检查和统一代码风格，为代码审阅等后续流程提供了自动化的可能。目前 PouchContainer 在 __<span data-type="color" style="color:#F5222D">每一次</span>__ 开发者提的 Pull Request 都会在 CircleCI 运行上述的代码检查工具。如果检查工具显示异常，代码审阅者有权 __<span data-type="color" style="color:#F5222D">拒绝</span>__ 审阅，甚至可以拒绝合并代码。

除了官方提供的工具外，我们还可以在开源社区中选择第三方的代码检查工具，比如 [errcheck](https://github.com/kisielk/errcheck) 检查开发者是否都处理了函数返回的 error 。但是这些工具并没有统一的输出格式，这很难完成不同工具输出结果的整合。好在开源社区有人实现了这一层统一的接口，即 [gometalinter](https://github.com/alecthomas/gometalinter)，它可以整合各种代码检查工具，推荐采用的组合是：

* [golint](https://github.com/golang/lint) - Google's (mostly stylistic) linter.
* [gofmt -s](https://golang.org/cmd/gofmt/) - Checks if the code is properly formatted and could not be further simplified.
* [goimports](https://godoc.org/golang.org/x/tools/cmd/goimports) - Checks missing or unreferenced package imports.
* [go vet](https://golang.org/cmd/vet/) - Reports potential errors that otherwise compile.
* [varcheck](https://github.com/opennota/check) - Find unused global variables and constants.
* [structcheck](https://github.com/opennota/check) - Find unused struct fields
* [errcheck](https://github.com/kisielk/errcheck) - Check that error return values are used.
* [misspell](https://github.com/client9/misspell) - Finds commonly misspelled English words.

每个项目都可以根据自己的需求来订制 gometalinter 套餐。

## 1.2 Shellcheck - 减少 shell script 潜在问题

shell script 虽然功能强大，但是它依然需要语法检查来避免一些潜在的、不可预判的错误。比如定义了未使用的变量，虽然不影响脚本的使用，但是它的存在会成为项目维护者的负担。

```powershell
#!/usr/bin/env bash

pouch_version=0.5.x

dosomething() {
  echo "do something"
}

dosomething
```

PouchContainer 会使用 [shellcheck](https://github.com/koalaman/shellcheck) 来检查目前项目里的 shell script。就以上述代码为例，shellcheck 检测会获得未使用变量的警告。该工具可以在代码审阅阶段发现 shell script 潜在的问题，减少运行时出错的概率。

```plain
In test.sh line 3:
pouch_version=0.5.x
^-- SC2034: pouch_version appears unused. Verify it or export it.
```

PouchContainer 当前的持续集成任务会扫描项目里 `.sh` 脚本，并逐一使用 shellcheck 来检查，详情请查看[这里](https://github.com/alibaba/pouch/blob/master/.circleci/config.yml#L21-L24)。

> NOTE: 当 shellcheck 检查太过于严格了，项目里可以通过加注释的方式来避开检查，或者是项目里统一关闭某项检查。具体的检查规则可查看[这里](https://github.com/koalaman/shellcheck/wiki)。

## 1.3 Markdownlint - 统一文档格式编排

PouchContainer 作为开源项目，它的文档同代码一样重要，因为文档是让用户了解 PouchContainer 的最佳方式。文档采用 markdown 的方式来编写，它的编排格式和拼写错误都是项目重点照顾对象。

同代码一样，光有文本约定还是会出现漏判，所以 PouchContainer 采用 [markdownlint](https://github.com/markdownlint/markdownlint) 和 [misspell](https://github.com/client9/misspell) 来检查文档格式和拼写错误，这些检查的地位同 `golint` 一样，会在每次 Pull Request 都会在 CircleCI 中运行，一旦出现异常，代码审阅者有权 __<span data-type="color" style="color:#F5222D">拒绝</span>__ 审阅或者合并代码。

PouchContainer 当前的持续集成任务会检查项目里的 markdown 文档编排格式，同时还检查了所有文件里的拼写，具体配置可查看[这里](https://github.com/alibaba/pouch/blob/master/.circleci/config.yml#L13-L20)。

> NOTE: 当 markdownlint 要求太过于严格时，项目里可以关闭相应的检查。具体的检查项目可查看[这里](https://github.com/markdownlint/markdownlint/blob/master/docs/RULES.md)。

## 1.4 小结

上述内容都属于风格纪律问题，PouchContainer 将编码规范检测自动化，集成到每一次的代码审阅中，帮助审阅者发现潜在的问题。

# 2. 如何编写 golang 的单元测试

单元测试可用来保证单一模块的正确性。在测试领域的金字塔里，单元测试覆盖面越广，覆盖功能越全，它就越能减少集成测试以及端到端测试所带来的调试成本。在复杂的系统里，任务处理的链路越长，定位问题的成本就越高，尤其是小模块所引发的问题。接下来的内容会分享 PouchContainer 编写 golang 单元测试用例的总结。

## 2.1 Table-Driven<span data-type="color" style="color:rgb(36, 41, 46)"><span data-type="background" style="background-color:rgb(255, 255, 255)"> </span></span>Test - DRY

简单地理解单元测试是给定某一个函数既定的输入，判断是否能得到预期的输出。当被测试的函数有各式各样的输入场景时，我们可以采用 Table-Driven 的形式来组织我们的测试用例，如接下来的代码所示。Table-Driven 采用数组的方式来组织测试用例，并通过循环执行的方式来验证函数的正确性。

```go
// from https://golang.org/doc/code.html#Testing
package stringutil

import "testing"

func TestReverse(t *testing.T) {
	cases := []struct {
		in, want string
	}{
		{"Hello, world", "dlrow ,olleH"},
		{"Hello, 世界", "界世 ,olleH"},
		{"", ""},
	}
	for _, c := range cases {
		got := Reverse(c.in)
		if got != c.want {
			t.Errorf("Reverse(%q) == %q, want %q", c.in, got, c.want)
		}
	}
}
```

为了方便调试和维护测试用例，我们可以加入一些辅助信息来描述当前的测试。比如 [reference](https://github.com/alibaba/pouch/blob/master/pkg/reference/parse_test.go#L54)  想要测试 [punycode](https://en.wikipedia.org/wiki/Punycode) 的输入时，如果不加入 `punycode` 的字样，对于代码审阅者或者项目维护者而言，他们可能不知道 `xn--bcher-kva.tld/redis:3` 和 `docker.io/library/redis:3` 之间的区别。

```go
{
		name:  "Normal",
		input: "docker.io/library/nginx:alpine",
		expected: taggedReference{
			Named: namedReference{"docker.io/library/nginx"},
			tag:   "alpine",
		},
		err: nil,
}, {
		name:  "Punycode",
		input: "xn--bcher-kva.tld/redis:3",
		expected: taggedReference{
			Named: namedReference{"xn--bcher-kva.tld/redis"},
			tag:   "3",
		},
		err: nil,
}
```

但是有些函数行为比较复杂，一次输入并不能作为一次完整的测试用例。例如 [TestTeeReader](https://github.com/golang/go/blob/release-branch.go1.9/src/io/io_test.go#L284) , TeeReader 从 buffer 里读出 <span data-type="color" style="color:rgb(3, 47, 98)"><span data-type="background" style="background-color:rgb(255, 255, 255)"><code>hello, world</code></span></span><span data-type="color" style="color:rgb(3, 47, 98)"><span data-type="background" style="background-color:rgb(255, 255, 255)"> </span></span><span data-type="background" style="background-color:rgb(255, 255, 255)">之后，已经将数据读取完毕了，如果再去读取，预期的行为是会遇到 end-of-file 的错误。这样的测试用例需要单独一个 case 来完成，不需要硬凑出 Table-Driven 的形式。</span>

简单来说，如果你测试某一个函数需要拷贝大部分代码时，理论上这些测试代码都可以抽出来，并使用 Table-Driven 的方式来组织测试用例，<strong><span data-type="color" style="color:#F5222D">Don`t Repeat Yourself</span></strong><span data-type="color" style="color:#F5222D"> </span>是我们遵守的原则。

> NOTE: Table-Driven 组织方式是 golang 社区所推荐，详情请查看[这里](https://github.com/golang/go/wiki/TableDrivenTests)。

## 2.2 Mock - 模拟外部依赖

在测试过程经常会遇到依赖的问题，比如 PouchContainer client 需要 HTTP server ，但这对于单元而言太重，而且这属于集成测试的范畴。那么该如何完成这部分的单元测试呢？

在 golang 的世界里，interface 的实现属于 [Duck Type](https://en.wikipedia.org/wiki/Duck_typing) 。某一个接口可以有各式各样的实现，只要实现能符合接口定义。如果外部依赖是通过 interface 来约束，那么单元测试里就模拟这些依赖行为。接下来的内容将分享两种常见的测试场景。

### 2.2.1 RoundTripper

还是以 PouchContainer client 测试为例。PouchContainer client 所使用的是 [http.Client](https://golang.org/pkg/net/http/#Client)。其中 http.Client 中使用了 [RoundTripper](https://golang.org/pkg/net/http/#RoundTripper) 接口来执行一次 HTTP 请求，它允许开发者自定义发送 HTTP 请求的逻辑，这也是 golang 能在原有基础上完美支持 HTTP 2 协议的重要原因。

```plain
http.Client -> http.RoundTripper [http.DefaultTransport]
```

对于 PouchContainer client 而言，测试关注点主要在于传入目的地址是否正确、传入的 query 是否合理，以及是否能正常返回结果等。因此在测试之前，开发者需要准备好对应的 RoundTripper 实现，该实现并不负责实际的业务逻辑，它只是用来判断输入是否符合预期即可。

如接下来的代码所示，PouchContainer `newMockClient` 可接受自定义的请求处理逻辑。在测试删除镜像的用例中，开发者在自定义的逻辑里判断了目的地址和 HTTP Method 是否为 DELETE，这样就可以在不启动 HTTP Server 的情况下完成该有的功能测试。

```go
// https://github.com/alibaba/pouch/blob/master/client/client_mock_test.go#L12-L22
type transportFunc func(*http.Request) (*http.Response, error)

func (transFunc transportFunc) RoundTrip(req *http.Request) (*http.Response, error) {
        return transFunc(req)
}

func newMockClient(handler func(*http.Request) (*http.Response, error)) *http.Client {
        return &http.Client{
                Transport: transportFunc(handler),
        }
}

// https://github.com/alibaba/pouch/blob/master/client/image_remove_test.go
func TestImageRemove(t *testing.T) {
        expectedURL := "/images/image_id"

        httpClient := newMockClient(func(req *http.Request) (*http.Response, error) {
                if !strings.HasPrefix(req.URL.Path, expectedURL) {
                        return nil, fmt.Errorf("expected URL '%s', got '%s'", expectedURL, req.URL)
                }
                if req.Method != "DELETE" {
                        return nil, fmt.Errorf("expected DELETE method, got %s", req.Method)
                }

                return &http.Response{
                        StatusCode: http.StatusNoContent,
                        Body:       ioutil.NopCloser(bytes.NewReader([]byte(""))),
                }, nil
        })

        client := &APIClient{
                HTTPCli: httpClient,
        }

        err := client.ImageRemove(context.Background(), "image_id", false)
        if err != nil {
                t.Fatal(err)
        }
}
```

### 2.2.2 MockImageManager

对于内部 package 之间的依赖，比如 PouchContainer Image API Bridge 依赖于 PouchContainer Daemon ImageManager，而其中的依赖行为由 interface 来约定。如果想要测试 Image Bridge 的逻辑，我们不必启动 containerd ，我们只需要像 RoundTripper 那样，实现对应的 Daemon ImageManager 即可。

```go
// https://github.com/alibaba/pouch/blob/master/apis/server/image_bridge_test.go
type mockImgePull struct {
        mgr.ImageMgr
        handler func(ctx context.Context, imageRef string, authConfig *types.AuthConfig, out io.Writer) error
}

func (m *mockImgePull) PullImage(ctx context.Context, imageRef string, authConfig *types.AuthConfig, out io.Writer) error {
        return m.handler(ctx, imageRef, authConfig, out)
}

func Test_pullImage_without_tag(t *testing.T) {
        var s Server

        s.ImageMgr = &mockImgePull{
                ImageMgr: &mgr.ImageManager{},
                handler: func(ctx context.Context, imageRef string, authConfig *types.AuthConfig, out io.Writer) error {
                        assert.Equal(t, "reg.abc.com/base/os:7.2", imageRef)
                        return nil
                },
        }
        req := &http.Request{
                Form:   map[string][]string{"fromImage": {"reg.abc.com/base/os:7.2"}},
                Header: map[string][]string{},
        }
        s.pullImage(context.Background(), nil, req)
}
```

### 2.2.3 小结

ImageManager 和 RoundTripper 除了接口定义的函数数目不同以外，模拟的方式是一致的。通常情况下，开发者可以手动定义一个将方法作为字段的结构体，如接下来的代码所示。

```go
type Do interface {
    Add(x int, y int) int
    Sub(x int, y int) int
}

type mockDo struct {
    addFunc func(x int, y int) int
    subFunc func(x int, y int) int
}

// Add implements Do.Add function.
type (m *mockDo) Add(x int, y int) int {
    return m.addFunc(x, y)
}

// Sub implements Do.Sub function.
type (m *mockDo) Sub(x int, y int) int {
    return m.subFunc(x, y)
}
```

当接口比较大、比较复杂的时候，手动的方式会给开发者带来测试上的负担，所以社区提供了自动生成的工具，比如 [mockery](https://github.com/vektra/mockery) ，减轻开发者的负担。

## 2.3 其他偏门

有些时候依赖的是第三方的服务，比如 PouchContainer client 就是一个很典型的案例。上文介绍 Duck Type 可以完成该案例的测试。除此之外，我们还可以通过注册 http.Handler 的方式，并启动 mockHTTPServer 来完成请求处理。这样测试方式比较重，建议在不能通过 Duck Type 方式测试时再考虑使用，或者是放到集成测试中完成。

> NOTE: golang 社区有人通过修改二进制代码的方式来完成 [monkeypatch](https://github.com/bouk/monkey) 。这个工具不建议使用，还是建议开发者设计和编写出可测试的代码。

## 2.4 小结

PouchContainer 将单元测试用例集成到代码审阅阶段，审阅者可以随时查看测试用例的运行情况。


# 3. <span data-type="color" style="color:rgb(34, 34, 34)"><span data-type="background" style="background-color:rgb(255, 255, 255)">总结</span></span>

在代码审阅阶段，应该通过持续集成的方式，将代码风格检查、单元测试和集成测试跑起来，这样才能帮助审阅者作出准确的决定，而目前 PouchContainer 主要通过 TravisCI/CircleCI 和 [pouchrobot](https://github.com/pouchcontainer/pouchrobot) 来完成代码风格检查和测试等操作。
