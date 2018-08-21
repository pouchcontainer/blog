# 0.Preface

 

As the function of  [PouchContainer](https://github.com/alibaba/pouch) continues to be iterated and refined, the project has grown in size, attracting a number of external developers to participate in the development of the project. Because each contributor's coding habits are different, the code reviewer's responsibility is not only to focus on logical correctness and performance issues, but also on code style, since a uniform code specification is a prerequisite for maintaining project code maintainability. In addition to unifying the project code style, the coverage and stability of test cases is also the focus of the project. In a nutshell, in the absence of regression test cases, how do you ensure that each code update does not affect existing functions?

This article shares PouchContainer's practices in code style specifications and golang unit test cases.

# 1.Unified coding style specification

PouchContainer is a project built by the golang language, which uses shell scripts to perform automated operations such as compiling and packaging. In addition to golang and shell scripts, PouchContainer also contains a large number of Markdown-style documents, which is the entry point for users to understand and understand PouchContainer. Its standard layout and correct spelling are also the focus of the project. The following sections will describe the tools and usage scenarios that PouchContainer uses in coding style specifications.

## 1.1 Golinter - Unicode format

Golang's grammar design is simple, and the community has a complete [CodeReview](https://github.com/golang/go/wiki/CodeReviewComments) guide from the beginning, so most of the golang projects have the same code style, and rarely fall into the unnecessary __religious__ dispute. On the basis of the community, PouchContainer also defines some specific rules to stipulate the developer. In order to ensure the readability of the code, the specific content can be read [here](https://github.com/alibaba/pouch/blob/master/docs/contributions/code_styles.md#additional-style-rules).

However, it is difficult to ensure that the project code style is consistent with the written protocol alone. Therefore, like other languages, golang provides the official tool chain, such as [golint](https://github.com/golang/lint), [gofmt](https://golang.org/cmd/gofmt), [goimports](https://github.com/golang/tools/blob/master/cmd/goimports/doc.go), and [go vet](https://golang.org/cmd/vet), which can be used to check and unify code styles before compilation, and to automate subsequent processes such as code review. may. Currently, PouchContainer runs the above code checking tool in CircleCI  __<span data-type="color" style="color:#F5222D">every</span>__ Pull Request submitted by the developer. If the inspection tool displays an exception, the code reviewer has the right to __<span data-type="color" style="color:#F5222D">refuse</span>__  the review and may even reject the merge code.

In addition to the official tools provided, we can also select third-party code inspection tools in the open source community, such as [errcheck](https://github.com/kisielk/errcheck) to check if the developer has processed the error returned by the function. However, these tools do not have a uniform output format, which makes it difficult to integrate the output of different tools. Fortunately, some people in the open source community have implemented this unified interface that is [gometalinter](https://github.com/alecthomas/gometalinter), which can integrate various code checking tools. The recommended combination is:

* [golint](https://github.com/golang/lint) - Google's (mostly stylistic) linter.
* [gofmt -s](https://golang.org/cmd/gofmt/) - Checks if the code is properly formatted and could not be further simplified.
* [goimports](https://godoc.org/golang.org/x/tools/cmd/goimports) - Checks missing or unreferenced package imports.
* [go vet](https://golang.org/cmd/vet/) - Reports potential errors that otherwise compile.
* [varcheck](https://github.com/opennota/check) - Find unused global variables and constants.
* [structcheck](https://github.com/opennota/check) - Find unused struct fields
* [errcheck](https://github.com/kisielk/errcheck) - Check that error return values are used.
* [misspell](https://github.com/client9/misspell) - Finds commonly misspelled English words.

Each project can customize the gometalinter package according to its own needs.

## 1.2 Shellcheck - Reduce potential problems with shell scripts

Although shell scripts are powerful, they still require syntax checking to avoid potential and unpredictable errors. For example, the definition of unused variables, although it does not affect the use of the script, its existence will become a burden on the project maintainer.

```powershell
#!/usr/bin/env bash

pouch_version=0.5.x

dosomething() {
  echo "do something"
}

dosomething
```

PouchContainer will use [shellcheck](https://github.com/koalaman/shellcheck) to check the shell script in the current project. Taking the above code as an example, shellcheck detection will get a warning of unused variables. This tool can detect potential problems with shell scripts during the code review phase, reducing the chance of runtime errors. 

```plain 
In test.sh line 3:
pouch_version=0.5.x
^-- SC2034: pouch_version appears unused. Verify it or export it.
```

PouchContainer's current continuous integration task scans the ` .sh` scripts in the project and checks them one by one using shellcheck, as shown [here](https://github.com/alibaba/pouch/blob/master/.circleci/config.yml#L21-L24).

> NOTE: When the shellcheck check is too strict, the project can be bypassed by a comment, or a check can be closed in the project. Specific inspection rules can be found [here](https://github.com/koalaman/shellcheck/wiki).

## 1.3 Markdownlint - uniform document formatting

PouchContainer is an open source project whose documentation is as important as the code, because documentation is the best way for users to understand PouchContainer. The document is written in the form of markdown, and its formatting and spelling errors are the focus of the project.

Like the code, there is a text convention or a missed judgment, so PouchContainer uses [markdownlint](https://github.com/markdownlint/markdownlint) and [misspell](https://github.com/client9/misspell) to check the document format and spelling errors. These checks have the same status as `golint` and will run in CircleCI every time Pull Request occurs. Code reviewers have the right to __<span data-type="color" style="color:#F5222D"> refuse</span>__  to review or merge the code Once an abnormality occurs.

PouchContainer's current continuous integration task checks the markdown document formatting in the project and also checks the spelling in all files. The configuration can be found [here](https://github.com/alibaba/pouch/blob/master/.circleci/config.yml#L13-L20).

> NOTE: When the markdownlint requirement is too strict, the corresponding check can be closed in the project. Specific inspection items can be found [here](https://github.com/markdownlint/markdownlint/blob/master/docs/RULES.md).

## 1.4 Summary

All of the above are style discipline issues, and PouchContainer automates code specification detection and integrates into each code review to help reviewers identify potential problems.

# 2. How to write a unit test for golang

Unit tests can be used to ensure the correctness of a single module. In the pyramid of test areas, the wider and more comprehensive the unit test coverage, the more it can reduce the debugging costs of integration testing and end-to-end testing.  The complex System have the longer the link processed by the task, the higher the cost of the location problem, especially the problems caused by small modules. The following sections share a summary of PouchContainer's preparation of golang unit test cases.

## 2.1 Table-Driven<span data-type="color" style="color:rgb(36, 41, 46)"><span data-type="background" style="background-color:rgb(255, 255, 255)"> </span></span>Test - DRY

A simple understanding of unit testing is to give a given input to a function to determine if the expected output can be obtained. When the function being tested has a variety of input scenarios, we can organize our test cases in the form of Table-Driven, as shown in the next code. Table-Driven uses arrays to organize test cases and validate the correctness of the function by loop execution.

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

To facilitate debugging and maintaining of test cases, we can add some auxiliary information to describe the current test. Such as the [reference](https://github.com/alibaba/pouch/blob/master/pkg/reference/parse_test.go#L54) to test the [punycode](https://en.wikipedia.org/wiki/Punycode) input, if you don't include the word punycode, they may not know the difference between `xn--bcher-kva.tld/redis:3` and `docker.io/library/redis: 3` for code reviewers or project maintainers.

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

However, some functions have complex behavior, and one input cannot be used as a complete test case. For example, [TestTeeReader](https://github.com/golang/go/blob/release-branch.go1.9/src/io/io_test.go#L284) and TeeReader read the  <span data-type="color" style="color:rgb(3, 47, 98)"><span data-type="background" style="background-color:rgb(255, 255, 255)"><code>hello, world</code></span></span><span data-type="color" style="color:rgb(3, 47, 98)"><span data-type="background" style="background-color:rgb(255, 255, 255)"> </span></span><span data-type="background" style="background-color:rgb(255, 255, 255)">  from the buffer, and the data has been read. If you read it again, the expected behavior is an end-of-file error. Such a test case needs to be done in a single case, without the need to hard-bake the form of Table-Driven. </span>

In simple terms, if you test a function that needs to copy most of the code, theoretically the test code can be extracted and used to organize test cases using Table-Driven. <strong><span data-type="color" style="color:#F5222D">Don`t Repeat Yourself</span></strong><span data-type="color" style="color:#F5222D"> </span>

> NOTE: The Table-Driven organization is recommended by the golang community. Please check [here](https://github.com/golang/go/wiki/TableDrivenTests) for details.

## 2.2 Mock - Simulating external dependence

There are often problems with dependencies during the testing process. For example, the PouchContainer client requires an HTTP server, but this is too heavy for the unit, and this is a category of integration testing. So how to complete this part of the unit test?

In the world of golang, the implementation of the interface belongs to [Duck Type](https://en.wikipedia.org/wiki/Duck_typing). An interface can have a variety of implementations, as long as the implementation conforms to the interface definition. If the external dependencies are constrained by the interface, then the dependency behavior is simulated in the unit test. The following content will share two common test scenarios. 

### 2.2.1 RoundTripper

Take the PouchContainer client test as an example. The PouchContainer client uses [http.Client](https://golang.org/pkg/net/http/#Client), http.Client uses the [RoundTripper](https://golang.org/pkg/net/http/#RoundTripper) interface to execute an HTTP request, which allows developers to customize the logic of sending HTTP requests. It is also an important reason why golang can perfectly support the HTTP 2 protocol on an original basis.

 ```plain
http.Client -> http.RoundTripper [http.DefaultTransport]
 ```

For the PouchContainer client, the test focus is mainly on whether the incoming destination address is correct, whether the incoming query is reasonable, and whether the result can be returned normally and so on. So before testing, developers need to prepare the corresponding RoundTripper implementation, which is not responsible for the actual business logic, it is only used to determine whether the input meets expectations or not.

As shown in the followings, PouchContainer `newMockClient` accepts customized request processing logic. In the test case of removing the image, the developer determines whether the destination address and the HTTP Method are DELETE in the customized logic, so that the functional test can be completed without starting the HTTP Server.

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

For dependencies between internal packages, such as the PouchContainer Image API Bridge depends on the PouchContainer Daemon ImageManager, and the dependency behavior is defined by interface. If we want to test the logic of Image Bridge, we don't have to start containerd, we just need to implement the corresponding Daemon ImageManager like RoundTripper.

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

ImageManager and RoundTripper are modeled in the same way except for the number of functions defined by the interface. In general, developers can manually define a structure that uses methods as fields, as shown in the followings.

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

When the interface is large and complex, the manual method will impose a test burden on the developer, so the community provides automatically generated tools, such as [mockery](https://github.com/vektra/mockery), to ease the burden on the developer.

## 2.3 Others

Sometimes it relies on third-party services, such as the PouchContainer client, which is a typical case. The above describes Duck Type to complete the test of this case. In addition, we can also complete the request processing by registering the http.Handler and starting mockHTTPServer. This way of testing is relatively heavy, which is not recommended to use until the Duck Type can not test, or put it into the integration test.

> NOTE: Someone in the golang community has accomplished [monkeypatch](https://github.com/bouk/monkey) by modifying the binary code. This tool is not recommended to use, it is still more advisable for developers to design and write testable code.

## 2.4 Summary

PouchContainer integrates unit test cases into the code review phase, and reviewers can view the running of test cases at any time.



# 3. Summary

In the code review phase, code style checking, unit testing, and integration testing should be run through continuous integration to help reviewers make accurate decisions. Currently, PouchContainer performs operations such as code style checks and tests primarily through TravisCI/CircleCI and [pouchrobot](https://github.com/pouchcontainer/pouchrobot). 