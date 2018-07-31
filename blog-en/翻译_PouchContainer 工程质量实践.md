# 0. Preface

With the development of [PouchContainer](https://github.com/alibaba/pouch), the functions are getting abundant and the project is getting vast, therefore many outside developer are attracted to join the developing of the project. Since the coding style of different person can be diverse while the unified coding style is the prerequisite of the code maintenance, the reviewer has the responsibility of not only checking the correctness and the  performance of the code but also unifying the style of the code. What's more, the coverage and the stability of the test cases are also important points requiring more attention. You can simply imagine how the project lacking the regression test case ensure that the update of the code won't affect the current function every time. 

In this passage, the coding style regulation exercise and the unit test case instance of golang will be explained. 

# 1. Unified coding style regulation 
PouchContainer is a project developed in golang where shell script is used to complete some automated operation, such as compilation and packaging. Besides golang and shell script, PouchContainer also contains loads of Markdown documents which are the entrance for users getting to know PouchContainer so that the regulated typesetting and the corrected spelling are also important aspects that need more focus. In the following chapter, the coding style regulation and the application scenario of PouchContainer will be introduced. 

## 1.1 Golinter - the unified coding style
The grammar of golong is designed simply and the community is full of [CodeReview](https://github.com/golang/go/wiki/CodeReviewComments) instruction so that most of the golang projects have the similar coding style and few of them will get stuck with the __religion__ fight. Basing on the community, PouchContainer also defines some specific rules to regulate developer to ensure the readability of their code. The details of the rules can be found [there](https://github.com/alibaba/pouch/blob/master/docs/contributions/code_styles.md#additional-style-rules).

But it's hard to ensure the unification of the project coding style simply relying on the written agreement. Therefore, like other languages, golang has official fundamental tool chain such as [golint](https://github.com/golang/lint),  [gofmt](https://golang.org/cmd/gofmt)，[goimports](https://github.com/golang/tools/blob/master/cmd/goimports/doc.go) and [go vet](https://golang.org/cmd/vet),etc. Those tools can be used to check and unify the coding style before compilation to make the automated coding review possible. For now,  PouchContainer will run above checking tools in CircleCI before __<span data-type="color" style="color:#F5222D">every</span>__ pull request raised by developer. If error is detected by those tools, the code reviewer has right to __<span data-type="color" style="color:#F5222D">refuse</span>__   examination or even refuse to merge those code.

Besides the provided official toolkit, we can use the code chekcing toolkit provided by the third party in the open source community as well, such as [errcheck](https://github.com/kisielk/errcheck) can check whether developer has eliminated all of the errors returned by functions. But those tools don't have the same output format, which make it difficult to merge the outputs produced by different tools. The good news is that someone in the community achieves the unified interface of this layer, which is [gometalinter](https://github.com/alecthomas/gometalinter). It can integrate various code checking toolkit, the recommended combinations are:

* [golint](https://github.com/golang/lint) - Google's (mostly stylistic) linter.
* [gofmt -s](https://golang.org/cmd/gofmt/) - Checks if the code is properly formatted and could not be further simplified.
* [goimports](https://godoc.org/golang.org/x/tools/cmd/goimports) - Checks missing or unreferenced package imports.
* [go vet](https://golang.org/cmd/vet/) - Reports potential errors that otherwise compile.
* [varcheck](https://github.com/opennota/check) - Find unused global variables and constants.
* [structcheck](https://github.com/opennota/check) - Find unused struct fields
* [errcheck](https://github.com/kisielk/errcheck) - Check that error return values are used.
* [misspell](https://github.com/client9/misspell) - Finds commonly misspelled English words.

Every project can order specific gometalinter combination basing on their needs.

## 1.2 Shellcheck - reduce potential problem in shell script

Although shell script has powerful functions, it still needs grammar check to avoid some potential and unpredictable error, such as the definition of the unused variable, which won't affect the normal function of the script but its existence will become the burden of the maintenance. 

```powershell
#!/usr/bin/env bash

pouch_version=0.5.x

dosomething() {
  echo "do something"
}

dosomething
```

PouchContainer will use [shellcheck](https://github.com/koalaman/shellcheck) to check the shell scripts in the current project. Taking the above code as an example, shellcheck will raise the warning that there is unused variable. This toll can detect the potential problems in shell script to reduce the error probability in running. 

```plain
In test.sh line 3:
pouch_version=0.5.x
^-- SC2034: pouch_version appears unused. Verify it or export it.
```

The current lasting integrating mission of PouchContainer will scan every `.sh` script in the project and use shellcheck to examine them one by one. The details can be found [there](https://github.com/alibaba/pouch/blob/master/.circleci/config.yml#L21-L24)。


> NOTE: When the examination of shellcheck is too strict, the project can use comment to avoid it, or shutdown some specific examination in the project. The detailed checking rule can be found [there](https://github.com/koalaman/shellcheck/wiki)。

## 1.3 Markdownlint - Unified document format

As an open-source project, documents have the same significance as code to PouchContainer. Because documents are the best methods for users get to know PouchContainer. Documents are written in markdown so that its formatting and spelling are mainly focused. 

Same as the code, there will be error missing merely relying on written aggrement. Therefore, PouchContainer applies [markdownlint](https://github.com/markdownlint/markdownlint) and [misspell](https://github.com/client9/misspell) to check document format and spelling. These examinations have the same reputation as `golint` which will be called in every Pull Request. Once an error is raised, the code reviewer has right to __<span data-type="color" style="color:#F5222D">refuse </span>__  inspect or merge those code.

The current lasting integration mission in PouchContainer will examine the markdown document formatting and document spelling. The specific configuration is showed [there](https://github.com/alibaba/pouch/blob/master/.circleci/config.yml#L13-L20).

> NOTE: 
> When the examination of mardownlint is too strict, the project can use comment to avoid it, or shutdown some specific examination in the project. The detailed checking rule can be found [there](https://github.com/markdownlint/markdownlint/blob/master/docs/RULES.md)。

## 1.4 Summary

The above content is all about the coding style problem. PouchContainer can check coding grammar automatically which is integrated in every time code review to help code reviewer find potential problems.  

# 2. How to make golang unit testing

Unit testing is used to guarantee the correctness of single module. In the testing pyramid, the broader the coverage of unit testing gets, the more general the covered functions become and the less testing cost generated by integration testing and end-to-end testing. In the complicated system, the longer the dealing chain of a mission is, the higher the cost of problem targeting gets, specially the problem caused by small modules. In the following chapters, the summary of how to develop golang unit testing will be introduced. 

## 2.1 Table-Driven<span data-type="color" style="color:rgb(36, 41, 46)"><span data-type="background" style="background-color:rgb(255, 255, 255)"> </span></span>Test - DRY
We can understand unit testing simply, that is giving a fixed input to some function and determine whether the output is expected. When the tested functions have lots of input scenarios, we could use Table-Driven to organize our test case, which is showed below. Table-Driven applies array to organize test case and uses loop to test the correctness of the function.  

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
For the convenience of testing and maintenance, we can add some aiding information to describe the current testing. For example, [reference](https://github.com/alibaba/pouch/blob/master/pkg/reference/parse_test.go#L54) wants to test the input of [punycode](https://en.wikipedia.org/wiki/Punycode), if there is no `punycode` like word, the code reviewer or project maintainer might not know the difference between `xn--bcher-kva.tld/redis:3` and `docker.io/library/redis:3`

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
But some fucntions have complicated behaviors, one input can't be taken as a completed test case. Take [TestTeeReader](https://github.com/golang/go/blob/release-branch.go1.9/src/io/io_test.go#L284) as an instance. After TeeReader reads <span data-type="color" style="color:rgb(3, 47, 98)"><span data-type="background" style="background-color:rgb(255, 255, 255)"><code>hello, world</code></span></span><span data-type="color" style="color:rgb(3, 47, 98)"><span data-type="background" style="background-color:rgb(255, 255, 255)"> </span></span><span data-type="background" style="background-color:rgb(255, 255, 255)"> from buffer, data reading is finished. If it tries to read it again, an error end-of-file will be raised. Test case like this needs a separate case to finish with no need to make Table-Drive format on purpose.  

</span>

Simply speaking, if you want to test some function with lots of code copied, those testing code can be withdrawn and use Table-Drive to organize test case.<strong><span data-type="color" style="color:#F5222D">Don`t Repeat Yourself</span></strong><span data-type="color" style="color:#F5222D"> </span> is the rule we obey. 

> NOTE: The organizing method of Table-Driven is recommended by golang community. The details can be found [there](https://github.com/golang/go/wiki/TableDrivenTests)。
## 2.2 Mock - Mock External Dependency

 During test procedure, we will always encounter dependency issues. For example, PouchContainer client needs HTTP server but it may over fit for units, and this belongs to integration test. So how can we complete this part of integration test?

In the world of golang, the realization of interface belongs to [Duck Type](https://en.wikipedia.org/wiki/Duck_typing). As long as the realization corresponds to the definition of the interface, there can be different realization at one interface. If external dependency constrains through interface, then the unit testing will mock these dependent behaviors. Here two common testing scenarios will be discussed.

### 2.2.1 RoundTripper

Take PouchContainer client testing as an example. PouchContainer client uses [http.Client](https://golang.org/pkg/net/http/#Client). http.Client [RoundTripper](https://golang.org/pkg/net/http/#RoundTripper) is used to  execute HTTP request， and it allows developer to customize the logic of sending HTTP request, and this is the reason why golang can support HTTP 2 contracts perfectly.

```plain
http.Client -> http.RoundTripper [http.DefaultTransport]
```

For PouchContainer client, the focus of testing is whether the target address is correct, whether incoming query is reasonable and whether the result can be returned normally and so on. Thus before testing, developer should have prepared corresponding RoundTripper realization, and this realization is not responsible for the real business logic. Instead it is only used to see if the input meets the expectation or not.

As the following codes shown, PouchContainer `newMockClient` can accept customized request processing logic. In the example of testing deleting mirror, developer will decide the target address and if the HTTP Method is DELETE in customized logic, and in this way the functional testing can be completed without HTTP Server being enabled. 

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

For dependency between internal packages (such as PouchContainer Image API Bridge is dependent on PouchContainer Daemon ImageManager), the dependent behavior is decided by interface. If we want to test the logic of Image Bridge, we do not need to enable containerd. Instead we just need to realize corresponding Daemon ImageManager, just like RoundTripper.

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

### 2.2.3 summary

ImageManager and RoundTripper mock in the same way except that the number of functions defined by the interface is different. In general, developer can define a structure in which the method is used as field, as the codes shown below.

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

When the interface is large and complex, a manual way will bring a burden to the testing, so our community provide tools that can automatically generate mocks for interfaces to release burden of developers, such as  [mockery](https://github.com/vektra/mockery).

## 2.3 others

Sometimes this relies on third party service, and a typical example would be PouchContainer client. Duck Type that we introduced above can be used to do this testing. Besides, we can also sign up for http.Handler and enable mockHTTPServer to complete the request of processing. But this testing method will over fit, and thus it is suggested that we use it only when Duck Type cannot achieve the goal or we use it during integration testing. 

> NOTE: In golang community some people complete  [monkeypatch](https://github.com/bouk/monkey) by adjusting binary codes. This tool is not suggested, and developers are suggested to  write codes which could be tested. 

## 2.4 summary

PouchContainer integrate unit testing case to code reviewing process, and reviewers can check the code performance of test case at any time. 

# 3. <span data-type="color" style="color:rgb(34, 34, 34)"><span data-type="background" style="background-color:rgb(255, 255, 255)">summary</span></span>

During code reviewing process, we should check codes, and run unit testing and integration testing to help reviewers to make the right decision. Now, PouchContainer mainly checks code and runs test through TravisCI/CircleCI 和 [pouchrobot](https://github.com/pouchcontainer/pouchrobot).


link: https://github.com/pouchcontainer/blog/blob/master/blog-cn/PouchContainer%20%E5%B7%A5%E7%A8%8B%E8%B4%A8%E9%87%8F%E5%AE%9E%E8%B7%B5.md


