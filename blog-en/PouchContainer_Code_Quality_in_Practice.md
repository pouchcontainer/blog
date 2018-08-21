# 0. Preface

As [PouchContainer](https://github.com/alibaba/pouch) is getting more and more stable and comprehensive, this scaling project has attracted many developers from different industries. As every developer's coding styles are quite different, the code reviewers have to take the responsibilities for not only ensuring performance and the correctness of logics, but more importantly, keeping a coherent coding style. A coherent coding style is always essential to a maintainable code base. Besides that, we may also pay attention to the test coverage rate as well as the application's stability. Think about this, how can we make sure the new code doesn't affect the existing features if we don't have a solid set of regression tests?

This article aims to share the recommended coding style for PouchContainer project, as well as some practical cases of unit testing in golong.

# 1. A coherent code style

PouchContainer is built with golang, and uses shell script for some automatic operations such as compling and packaging. Besides golang and shell script, PouchContainer contains a bunch of Markdown-style documents serving as an entry point for users who would like to dive into PouchContainer. The maintianers of this project also attach great importance to the proper format and correct spelling of the documents aforementioned.

## 1.1 Golinter - the Linter for coherent code style

golong's syntax is simple by designed, plus there are lots of [Code Review](https://github.com/golang/go/wiki/CodeReviewComments) guides in the community, it makes almost all golang projects have a similar code style, and this hence avoids lots of meaningless arguments. Besides the community, PouchContainer also defines some specific rules to ensure styles among developers, which benefits the readability and maintainability of the code. For more details, please have a look [here](https://github.com/alibaba/pouch/blob/master/docs/contributions/code_styles.md#additional-style-rules).

However, the style is still not perfectly coherent by only defining conventions, so just like many other languages, golang officially offers some linters, including [golint](https://github.com/golang/lint), [gofmt](https://golang.org/cmd/gofmt), [goimports](https://github.com/golang/tools/blob/master/cmd/goimports/doc.go), [go vet](https://golang.org/cmd/vet) and so on. These tools check the coding style before compling and make the following automatic procedures for code review possible. Currently, PouchContainer __goes through the code for <span data-type="color" style="color:#F5222D">each and every</span> pull request using CircleCI, and if the code doesn't pass, the code reviewer can <span data-type="color" style="color:#F5222D">reject to review</span> or even reject the PR__.

Apart from those tools officially provided, we can also pick one third-party tool in open-source community as well, these tools include [errcheck](https://github.com/kisielk/errcheck), which checks whether or not developers handle the error functions potentially return. Although these tools do not have a coherent output format which makes it hard to compose different tools, there exists some interface to hide the difference of the output formats, and that's [gometalinter](https://github.com/alecthomas/gometalinter). It successfully composes different linters and checkers, and here's the composition we recommend:

* [golint](https://github.com/golang/lint) - Google's (mostly stylistic) linter.
* [gofmt -s](https://golang.org/cmd/gofmt/) - Checks if the code is properly formatted and could not be further simplified.
* [goimports](https://godoc.org/golang.org/x/tools/cmd/goimports) - Checks missing or unreferenced package imports.
* [go vet](https://golang.org/cmd/vet/) - Reports potential errors that otherwise compile.
* [varcheck](https://github.com/opennota/check) - Find unused global variables and constants.
* [structcheck](https://github.com/opennota/check) - Find unused struct fields
* [errcheck](https://github.com/kisielk/errcheck) - Check that error return values are used.
* [misspell](https://github.com/client9/misspell) - Finds commonly misspelled English words.

Each project may have different gometalinter composition accordingly.

## 1.2 Shellcheck - Reduce the potential issues of shell script

Although shell script is powerful, but it's still necessary to apply syntax checking to avoid some potential and unpreditable problems. The checking will show warnings, such as unused variable, when syntax rules are broken, even if this doesn't affect the functionalities of the script but it does become a burden for maintainers.

```powershell
#!/usr/bin/env bash

pouch_version=0.5.x

dosomething() {
  echo "do something"
}

dosomething
```

PouchContainer will use [shellcheck](https://github.com/koalaman/shellcheck) to inspect the shell script in the project. Use the above code as an example, shellcheck will show an unused variable warning. This tool helps us discover potential issues of shell script and hence it reduces the possibility of crashing in runtime.

```plain
In test.sh line 3:
pouch_version=0.5.x
^-- SC2034: pouch_version appears unused. Verify it or export it.
```

PouchContainer's continuous integration tasks will scan `.sh` script in the project, and use shellcheck to go through them one by one. For more details, please have a look [here](https://github.com/alibaba/pouch/blob/master/.circleci/config.yml#L21-L24).

> Note: We can pass the check manually by adding comments in the code, if shellcheck is just too strict. For detailed rules for checking, please have a look [here](https://github.com/koalaman/shellcheck/wiki).

## 1.3 Markdownlint - the Linter for coherent document style

As an open-source project, PouchContainer's document is of the same level of importance as code, since document is the best way if someone wants to dive into PouchContainer. The document is written in Markdown format, and maintainers always ensure the format and spelling of words in it are correct.

Just like codes, text files could also have some issues that are hard to notice, so we use [markdownlint](https://github.com/markdownlint/markdownlint) and [misspell](https://github.com/client9/misspell) to ensure the coherent style and avoid misspelling. We also consider these checks the same important as `golint`, they will also be gone through in CircleCI for each PR. And similarly, once there's any error, code reviewers can __<span data-type="color" style="color:#F5222D">reject</span>__ to review or merge the pull request.

Continuous integration tasks in PouchContainer will check all markdown files' format and spelling. For the details regarding its configuration, you may have a look [here](https://github.com/alibaba/pouch/blob/master/.circleci/config.yml#L13-L20).

> Note: You can turn off the markdownlint checking in the project if it's too strict. [Here's a list of stuff it checks](https://github.com/markdownlint/markdownlint/blob/master/docs/RULES.md).

## 1.4 Summary

Developers are highly recommended to follow all the rules and conventions mentioned above. PouchContainer automates the checking and linting procedure, and enforces it in each time of code review in order to help the reviewers to notice potential issues at the early stage.

# 2. How to write unit test with golang

Unit test can be used to assure the accuracy of a single module. In the pyramid of test field, the wider of unit test coverage, the more complete of its function, the more it can reduce the debug cost of integration test and end-to-end test. In complex system, the longer the task processing link, the higher cost of positioning problem, especially in the problems caused by small module. The following content will share the summary of writting golang unit test cases in PouchContainer.

## 2.1 Table-Driven<span data-type="color" style="color:rgb(36, 41, 46)"><span data-type="background" style="background-color:rgb(255, 255, 255)"> </span></span>Test-DRY

Simple understanding of unit tests is given a certain input of a function, and determine whether can get the expected output. When the tested function has a wide variety of input scene, we can use the Table-Driven to form the organization of test cases, which is shown in the following code. Table-Driven uses array to organize test cases, and proves the validity of functions through the loop execution way.

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

In order to debug and maintain test cases easily, we can add some auxiliary information to describe current test. For example [reference](https://github.com/alibaba/pouch/blob/master/pkg/reference/parse_test.go#L54) wants to test the input of [punycode](https://en.wikipedia.org/wiki/Punycode) . If there is no `punycode`, the code reviewers or the project maintainer may not know the difference between `xn--bcher-kva.tld/redis:3` and `docker.io/library/redis:3`.

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

But behavior of some function is very complex, an input can not be as a complete test cases. Such as [TestTeeReader](https://github.com/golang/go/blob/release-branch.go1.9/src/io/io_test.go#L284) . After TeeReader read <span data-type="color" style="color:rgb(3, 47, 98)"><span data-type="background" style="background-color:rgb(255, 255, 255)"><code>hello, world</code></span></span><span data-type="color" style="color:rgb(3, 47, 98)"><span data-type="background" style="background-color:rgb(255, 255, 255)"> </span></span><span data-type="background" style="background-color:rgb(255, 255, 255)"> from buffer, the data is read out. If read again, it will encounter end-of-file error. This test cases need a single case to complete without the form of Table-Driven.</span>

In simple terms, if you need to copy most of the code when test a function, the test code can be drawn out in the theory, and you can organize test cases with Table-Driven, The principle that we need follow is <strong><span data-type="color" style="color:#F5222D">Don`t Repeat Yourself</span></strong><span data-type="color" style="color:#F5222D"> </span>.

> NOTE: The organization of Table-Driven is recommended by golang community. For more details, please check [here](https://github.com/golang/go/wiki/TableDrivenTests).

## 2.2 Mock - simulate external dependencies

There are also dependence problems during testing, such as the PouchContainer client requiring HTTP server, but this is too heavy for unit test, and it belongs to integration testing. So how do we complete this unit test?

In the world of golang, the implementation of interface belongs to [Duck Type](https://en.wikipedia.org/wiki/Duck_typing) . An interface can be implemented in a variety of ways, as long as the implementation accords the interface definition. If the external constraints are constrained by interface, then these dependence behaviors can be simulated by unit test. The next content will share two common test scenarios.

### 2.2.1 RoundTripper

Let's take the PouchContainer client test as an example. PouchContainer client uses [http.Client](https://golang.org/pkg/net/http/#Client) . And http.Client uses [RoundTripper](https://golang.org/pkg/net/http/#RoundTripper) to perform a HTTP request. It allows developers to customize the logic of sending HTTP requests, which is also an important reason for golang to fully support the HTTP 2 protocol on the original basis.

```plain
http.Client -> http.RoundTripper [http.DefaultTransport]
```

For PouchContainer client, the concerns of test are whether the input destination address is correct or the input query is reasonable, and whether the results can be returned properly. So before the test, the developer needs to make the corresponding RoundTripper implementation ready, which is not responsible for the actual business logic. It is only used to determine whether the input is in line with the expectations.

As the next code shows, PouchContainer `newMockClient` can accept custom request processing logic. In the case of deleting mirror, the developer determines whether the destination address and the HTTP Method are DELETE in the custom logic, so that the function test can be completed without starting HTTP Server.

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

For the dependence between the internal package, such as the PouchContainer Image API Bridge is dependent on the PouchContainer Daemon ImageManager, and the dependency behavior in it is convened by interface. If we want to test the logic of Image Bridge, we don't have to start containerd, but just need to implement Daemon ImageManager like RoundTripper.

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

### 2.2.3 Summary

Besides the number of functions defined by interfaces are different between ImageManager and RoundTripper, the way of simulation is consistent. Usually, developers can manually define a structure that uses the method as a field, as shown in the next code.

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

When the interface is larger and more complex, the manual way will bring burden to the developer, so the community provides automatic generation tools to reduce the burden of developers, such as [mockery](https://github.com/vektra/mockery) .

## 2.3 Other branch of learning

Sometimes relying on third party services, the PouchContainer client is a typical case. Duck Type can be used to complete the test. In addition, we can register the http. Handler and start mockHTTPServer to complete the request processing. This is a heavy way to test, and it is recommended that you consider it when you can't pass the Duck Type test, or put it into the integration test.

> NOTE: Someone in the golang community has done it by modifying binary code. [monkeypatch](https://github.com/bouk/monkey). It is recommended that developers design and write testable code instead of using this tool.

## 2.4 Summary

When PouchContainer integrates unit test cases into the code review phase, the reviewer can check how the test cases are running at any time.


# 3. <span data-type="color" style="color:rgb(34, 34, 34)"><span data-type="background" style="background-color:rgb(255, 255, 255)">Summary</span></span>

In the code review phase, code style checks, unit tests, and integration tests should be run in a continuous integration way to help the reviewers make accurate decisions. At present, PouchContainer completes code style check and test operation mainly through TravisCI/CircleCI and  [pouchrobot](https://github.com/pouchcontainer/pouchrobot) .

