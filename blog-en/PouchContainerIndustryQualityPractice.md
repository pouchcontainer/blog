# 0.Prologue
As [PouchContainer](https://github.com/alibaba/pouch)'s feature continuously evolve and complete, this attracted many outside developers to join the development of this project. Since the coding style and habit of each contributor is different, the code reviewer's responsibility is not only focused on logic correctness and performance issues, but also coding style, because a unified coding style is the prerequisite of ensuring the code's ease of maintenance. Furthermore the coverage and stability of the test is also one of the key metrics the project is focusing. Let's think about it, in a project without regression testing, how could we ensure each code update won't affect present features.

In this blog we will share the practices PouchContainer used in coding style guide and golang unit testing.

# 1.Unified Coding Style
PouchContainer is a project built upon golang, in this project shell scripts will be used to complete some automatic process, like compiling and packaging. Furthermore, PouchContainer also contains many Markdown style documentation, it's user's entrance, its formatting correctness and spelling correctness is also something the project's focus point. The following content will introduce the tools used in PouchContainer's Coding Style Guide and their related usage scenario.

## 1.1 Golinter - Unified Coding Style
Golang has a easy-to-use syntax design, plus the community has complete [CodeReview](https://github.com/golang/go/wiki/CodeReviewComments) guidelines from the very beginning, this makes most golang projects have the same coding style, and very unlikely to fall into some meaningless __religion__ arguments. On top of the community's existing foundation, PouchContainer defined some extra specific rules to constrain the developers in order to ensure the code's readability, for detail please read [Here](https://github.com/alibaba/pouch/blob/master/docs/contributions/code_styles.md#additional-style-rules)

But enforcing coding style based on documentations only is hard to keep coding styles consistent, so golang, like other languages, officially provided basic toolchains like [golint](https://github.com/golang/lint),  [gofmt](https://golang.org/cmd/gofmt)，[goimports](https://github.com/golang/tools/blob/master/cmd/goimports/doc.go) and [go vet](https://golang.org/cmd/vet),etc. This tools can check and unify coding styles prior to compiling, which in turn provides automation possibility for further operations like code review. Currently PouchContainer runs the aforementioned tools on CirclesCI __<span data-type="color" style="color:#F5222D">for each</span>__ Pull Request. If the syntax check tools shows that something is wrong, the code reviewer has the rights to reject reviewing or even reject merging the code.

Besides official tools, we can also choose third party tools provided by the community, for example [errcheck](https://github.com/kisielk/errcheck) checks if the developer has taken care of all the error returned by functions. However these tools doesn't have a unified output format, which makes it hard to combine the output of various tools. Fortunately someone from the opensource community provided a unified layer [gometalinter](https://github.com/alecthomas/gometalinter) which can combine all kinds of code check tools. The recommended toolset is:

* [golint](https://github.com/golang/lint) - Google's (mostly stylistic) linter.
* [gofmt -s](https://golang.org/cmd/gofmt/) - Checks if the code is properly formatted and could not be further simplified.
* [goimports](https://godoc.org/golang.org/x/tools/cmd/goimports) - Checks missing or unreferenced package imports.
* [go vet](https://golang.org/cmd/vet/) - Reports potential errors that otherwise compile.
* [varcheck](https://github.com/opennota/check) - Find unused global variables and constants.
* [structcheck](https://github.com/opennota/check) - Find unused struct fields
* [errcheck](https://github.com/kisielk/errcheck) - Check that error return values are used.
* [misspell](https://github.com/client9/misspell) - Finds commonly misspelled English words.

Each project can customize the ``gometalinter`` toolset based on their needs

## 1.2 Shellcheck - Reducing potential issues of the shell script
While shellscript is powerful, it still needs syntax check to avoid some potential, unexpected errors. For example unused variables, while they doesn't affect the execution of the script, but its existence will become a burden for maintainers.

```powershell
#!/usr/bin/env bash

pouch_version=0.5.x

dosomething() {
  echo "do something"
}

dosomething
```

PouchContainer will use [shellcheck](https://github.com/koalaman/shellcheck) to check shell scripts in the current project. Use the aforementioned code as a example, shellcheck check will results in a warning about unused variable.This tools could find potential issues of the shell script during code review stage and reduce the possibility of runtime error.

```plain
In test.sh line 3:
pouch_version=0.5.x
^-- SC2034: pouch_version appears unused. Verify it or export it.
```

PouchContainer's current continuous integration will scan for all the ``.sh`` scripts and use shellcheck to check them, for details please read [Here](https://github.com/alibaba/pouch/blob/master/.circleci/config.yml#L21-L24)

> NOTE: When shellcheck is being to strict, the project could avoid the check by adding comments, or globally turning off one of the checks. For detailed checker rules please read [here](https://github.com/koalaman/shellcheck/wiki)

## 1.3 Markdownlint - Unified Documentation Layout

As an open-source project, PouchContainer's documentation is as important as its code, because documentation is the best way for users to know about PouchContainer. The documentation is written in Markdown, its misspelling and misformating are both key focus points of the project.

Just like code, merely using text constraints also results in misjudgement, so PouchContainer uses [markdownlint](https://github.com/markdownlint/markdownlint) and [misspell](https://github.com/client9/misspell) to check for formatting and spelling errors, these checks are as important as  `golint` , and will be run in CircieCI for each Pull Request. The code reviewer has the rights to __<span data-type="color" style="color:#F5222D">reject</span>__ reviewing or merging the code if something is wrong

PouchContainer's current CI jobs will checks for all the formatting issues in Markdown documentations, as well as all spelling issues, for detailed configuration please check [here](https://github.com/alibaba/pouch/blob/master/.circleci/config.yml#L13-L20)。

> NOTE: When markdownlint is being to strict, the project could globally turning off one of the checks. For detailed checker rules please read [here](https://github.com/markdownlint/markdownlint/blob/master/docs/RULES.md)。

## 1.4 Summary

The aforementioned contents are all style and guideline issues, PouchContainer automates all style and guideline checks and intergrate them into each code review to help reviews find potential issues.  

# 2. How to write the unit test of golang

Unit test can be used to ensure the correctness of a single module. In the pyramid of test areas, the wider the unit test coverage is and the more comprehensive coverage is, the more it can reduce the debugging costs of integration testing and end-to-end testing. In a complex system, the longer the link processed by the task is, the higher the cost of the location problem will be, especially the problems caused by small modules. The following sections will share a summary of PouchContainer's preparation of golang unit test cases.

## 2.1 Table-Driven<span data-type="color" style="color:rgb(36, 41, 46)"><span data-type="background" style="background-color:rgb(255, 255, 255)"> </span></span>Test - DRY

To simplily understand of unit testing is to give a given input to a function to determine whether the expected output can be obtained. When the function being tested has a variety of input scenarios, we can organize our test cases in the form of Table-Driven, as code shown below. Table-Driven uses arrays to organize the test cases and validate the correctness of the function by loop execution.

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

To debugging and maintain of test cases easlily, we can add some auxiliary information to describe the current test. For example [reference](https://github.com/alibaba/pouch/blob/master/pkg/reference/parse_test.go#L54) want to test [punycode](https://en.wikipedia.org/wiki/ Punycode) input, if you do not add the word `punycode`, for code reviewers or project maintainers, they may not know the difference bewteen `xn--bcher-kva.tld/redis:3` and `docker.io/library/redis:3`.


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

However, some functions are more complicated, and one input cannot be used as a complete test case. For example [TestTeeReader] (https://github.com/golang/go/blob/release-branch.go1.9/src/io/io_test.go#L284) , TeeReader read <span data-type="color" style="color:rgb(3, 47, 98)"><span data-type="background" style="background-color:rgb(255, 255, 255)"><code>hello, world</code></span></span><span data-type="color" style="color:rgb(3, 47, 98)"><span data-type="background" style="background-color:rgb(255, 255, 255)"> </span></span><span data-type="background" style="background-color:rgb(255, 255, 255)"> from buffer. After data being read, if you read it again, the expected behavior is meeting an end-of-file error. Such a test case needs to be done in a single case, which do not need to hard-bake the form of Table-Driven. </span>

In simple terms, if you test a function that needs to copy the most of the code, in theory, the test code can be extracted, and use Table-Driven to organize the test case. <strong><span data-type="color" style="color:#F5222D">Don`t Repeat Yourself</span></strong><span data-type="color" style="color:#F5222D"> </span> is our principle.

> NOTE: The organization method of Table-Driven is recommended by the golang community. See [here](https://github.com/golang/go/wiki/TableDrivenTests) for details.


## 2.2 Mock - Simulating external dependencies

Some dependence problems are often meet during the testing process. For example, the PouchContainer client requires an HTTP server, but this is too heavy for the unit, and this is a category of integration testing. So how to complete this part of the unit test?

In the world of golang, the implementation of the interface belongs to [Duck Type](https://en.wikipedia.org/wiki/Duck_typing). An interface can have a variety of implementations, as long as the implementation conforms to the interface definition. If the external dependence are constrained by the interface, then the dependence behavior is simulated in the unit test. The following content will share two common test scene.

### 2.2.1 RoundTripper

Take the PouchContainer client test as an example. The PouchContainer client uses [http.Client](https://golang.org/pkg/net/http/#Client). The [RoundTripper](https://golang.org/pkg/net/http/#RoundTripper) interface is used in http.Client to perform an HTTP request, which allows developers to customize the logic for sending HTTP requests. This is also an important reason for golang to support the HTTP 2 protocol perfectly on the original basis.

```plain
http.Client -> http.RoundTripper [http.DefaultTransport]
```

For the PouchContainer client, the test focus is mainly on whether the incoming destination address is correct, whether the incoming query is reasonable, and whether the result can be returned normally. So before testing, developers need to prepare the corresponding RoundTripper implementation, which is not responsible for the actual business logic, but only used to determine whether the input meets expectations.

As the next code shown below, PouchContainer `newMockClient` accepts customized request processing logic. In the testing case of removing the mimirror, the developer determines in the customized logic whether the destination address and the HTTP Method are DELETE, then the functional test can be completed without starting the HTTP Server.


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

For dependencies between internal packages, for example, PouchContainer Image API Bridge depends on the PouchContainer Daemon ImageManager, in which the dependence behaviors are defined by interface. If we want to test the logic of Image Bridge, we don't have to start containerd, we just need to implement the corresponding Daemon ImageManager like RoundTripper.


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

ImageManager and RoundTripper

Besides the number of functions defined by the interface are different, the simulation ways are consistent. In general situlations, developers can manually defined a structure as fields by using of methods, shown in code below.


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

When the interface is large and complex, the manual method will impose a test burden on the developer, so the community provides automatic generation tools, such as [mockery](https://github.com/vektra/mockery), for easeing the burden on developers.

## 2.3 Other methods

Sometimes it depends on third-party services, for exapmle, the PouchContainer client, which is a typical case. The Duck Type described above can complete the test of this case. In addition, we can also complete the request processing by registering the http.Handler and starting mockHTTPServer. This testing is relatively heavy, it is recommended to consider the use of it when the Duck Type test is not possible, or put it into the integration test.

> NOTE: Someone in the golang community has done [monkeypatch](https://github.com/bouk/monkey) by modifying the binary code. This tool is not recommended, we still recommend developers to design and write testable code.

## 2.4 Summary

PouchContainer integrates unit test cases into the code review phase, and reviewers can view the running of test cases at any time.

# 3. <span data-type="color" style="color:rgb(34, 34, 34)"><span data-type="background" style="background-color:rgb(255, 255, 255)">Summary</span></span>

During the code revieW, code style checking, unit testing, and integration testing should be run through contin
