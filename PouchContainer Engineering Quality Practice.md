# 0. Abstract
With functions become iterated and complete, [PouchContainer](https://github.com/alibaba/pouch) has grown up gradually. Lots of external developers joined in and contributed to the project. Because each developer's coding styles are different, the reviewer's responsibility becomes difficult. It needs not only to focus on the logical accuracy and benchmarks, the coding styles are important because a unified coding rule helps to keep the project easy to maintain. In addition to a unified coding style, the coverage and stability of test cases are also key points under focus. We can't image how to ensure each update does not affect the existing features without regression of test cases.

The paper will talk about the standardization of coding style and test cases in golang units.

# 1. Rule of standard coding style
PouchContainer is written by golang language. There are some shell scripts to achieve automated operations such as compiling and packaging. In addition to golang and shell script, PouchContainer also has rich documents with Markdown style. They are the entries to know and learn for users. The standard composing and correct spelling are also in the spotlight. The following part will introduce some tools and situations used for specifications of PouchContainer's coding style.

## 1.1 Golinter - Uniforming of coding formats
golang has a clean grammar. As well as the complete community guide of [CodeReview](https://github.com/golang/go/wiki/CodeReviewComments) during the preliminary period, most codes have the similar style and are rarely drown into __religious__ conflicts. Based on community, PouchContainer also defined some particular rules to keep developers within bounds in order to maintain the readability of codes. Click [here](https://github.com/alibaba/pouch/blob/master/docs/contributions/code_styles.md#additional-style-rules) to read details.

However, it is not convincing to keep codes in a similar style just rely on written agreements. Thus, like other languages, it officially provides a set of fundamental tools, such as [golint](https://github.com/golang/lint), [gofmt](https://golang.org/cmd/gofmt), [goimports](https://github.com/golang/tools/blob/master/cmd/goimports/doc.go), [go vet](https://golang.org/cmd/vet) and etc. These tools are used to check and uniform coding styles, which provides automation possibilities for later processes such as code reviewing. Now, PouchContainer would run these code checking tools on CircleCI __<span data-type="color" style="color:#F5222D">for each</span>__  Pull Request asked by developers. If exceptions happen, reviewers have the right to __<span data-type="color" style="color:#F5222D">refuse</span>__ to check or even merge.

Besides official tools, we are able to choose third party's checking tools in open source communities. For example, [errcheck](https://github.com/kisielk/errcheck) checks whether developers have issued the error returned by functions. But these tools don't have a standard output format so that it is hard to achieve perfect integration of results from different tools. Fortunately, someone has designed this kind of standard interface in open source community -- [gometalinter](https://github.com/alecthomas/gometalinter). It is used to pull together kinds of code checking tools. The recommandation combination is:

* [golint](https://github.com/golang/lint) - Google's (mostly stylistic) linter.
* [gofmt -s](https://golang.org/cmd/gofmt/) - Checks if the code is properly formatted and could not be further simplified.
* [goimports](https://godoc.org/golang.org/x/tools/cmd/goimports) - Checks missing or unreferenced package imports.
* [go vet](https://golang.org/cmd/vet/) - Reports potential errors that otherwise compile.
* [varcheck](https://github.com/opennota/check) - Find unused global variables and constants.
* [structcheck](https://github.com/opennota/check) - Find unused struct fields
* [errcheck](https://github.com/kisielk/errcheck) - Check that error return values are used.
* [misspell](https://github.com/client9/misspell) - Finds commonly misspelled English words.
 
 Each project is able to get a customized package according to its demands.
 
 ## 1.2 Shellcheck - Reducing potential problems of shell script
Although shell script has powerful functions, it still needs grammar checking to avoid some potential and unknown errors such as definition of unused variables. It does not affect normal uses of scripts, but can be a burden of project maintainers.
 
```powershell
#!/usr/bin/env bash

pouch_version=0.5.x

dosomething() {
  echo "do something"
}

dosomething
```

PouchContainer uses [shellcheck](https://github.com/koalaman/shellcheck) to check the existing shell scripts in project. Using codes above as an example, shellcheck will get warning that comes from the unused variable. It is able to find potential problems at the reviewing time, which reduces the possibility of errors.

```plain
In test.sh line 3:
pouch_version=0.5.x
^-- SC2034: pouch_version appears unused. Verify it or export it.
```

Current continuous integration tasks of PouchContainer scan `.sh` scripts in project and check them off by shellcheck. Click [here](https://github.com/alibaba/pouch/blob/master/.circleci/config.yml#L21-L24) to see details.

> NOTE: If you feel shellcheck's checking is too serious, it's possible to avoid checking by injecting annotations or closing a certain type of checking in the whole project. Detailed checking rules can be found [here](https://github.com/koalaman/shellcheck/wiki).

## 1.3 Uniforming document format arrangement
As an open source project, PouchContainer's documentation is as important as code because documentation is the best way to let users know about PouchContainer. The document is written in markdown way. Its layout format and spelling mistakes are the key objects of the project.

Like the code, there is a text convention or a missing sentence, so PouchContainer uses [markdownlint](https://github.com/markdownlint/markdownlint) and [misspell](https://github.com/client9/misspell) to check the document format and spelling errors. These checks, like `golint` , will run in CircleCI each time in Pull Request, and the code reviewers have the right to __<span data-type="color" style="color:#F5222D">refuse</span>__ reading or merging codes once the anomaly occurs.

The current continuous integration task of PouchContainer will check the markdown document layout format in the project, and also check the spelling in all the files. The specific configuration can be seen [here](https://github.com/alibaba/pouch/blob/master/.circleci/config.yml#L13-L20).

> NOTE: When markdownlint is too strict, the corresponding checks can be closed in the project. Click [here] to see more details.(https://github.com/markdownlint/markdownlint/blob/master/docs/RULES.md)

## 1.4 Summary
All of the above are style discipline issues. PouchContainer automates code checking and integrates into each code reviewing to help reviewers identify potential problems.

# 2. How to wirte an unit testing of golang
Unit testings are used to ensure the correctness of unit modules. In the pyramid of testing areas, the wider and the more comprehensive unit testing covers, the more it can reduce debugging costs of integration testing and end-to-end testing. In a complex system, the longer a link processed by task, the higher the cost of location problem, especially the problems caused by small modules. The following sections share a summary of PouchContainer's composing of golang unit testings.

## 2.1 Table-Driven<span data-type="color" style="color:rgb(36, 41, 46)"><span data-type="background" style="background-color:rgb(255, 255, 255)"> </span></span>Test - DRY

We can easily consider unit testing as an input of a certain function and to see if we can get the expected output. When task function has a variety of input scenarios, we are able to organize our test cases in the form of Table-Driven, as shown in the following codes. Table-Driven uses arrays to organize test cases and validate the correctness of the function by loop execution.

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

For the convenience of testing and maintaining test cases, we could add some helpful information to describe current tests. For instance, if [reference](https://github.com/alibaba/pouch/blob/master/pkg/reference/parse_test.go#L54) wants to test the input of [punycode](https://en.wikipedia.org/wiki/Punycode) without `punycode` word, it may not be known to code reviewers and project maintainers that the difference between `xn -- bcher-kva.tld/redis:3` and `docker.io/library/redis:3`.

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

However, some actions of function are complex. One input is not considered as a complete test case such as [TestTeeReader](https://github.com/golang/go/blob/release-branch.go1.9/src/io/io_test.go#L284). TeeReader has already get the data after reading <span data-type="color" style="color:rgb(3, 47, 98)"><span data-type="background" style="background-color:rgb(255, 255, 255)"><code>hello, world</code></span></span><span data-type="color" style="color:rgb(3, 47, 98)"><span data-type="background" style="background-color:rgb(255, 255, 255)"> </span></span><span data-type="background" style="background-color:rgb(255, 255, 255)"> from buffer. It would cause errors like end-of-life when you read again. This kind of test cases needs an independent case instead of a cobbled format of Table-Driven. </span>

In another world, if you want to test codes with amounts of copings, these testing codes are able to get from it ideally and being tested by Table-Driven's test cases. <strong><span data-type="color" style="color:#F5222D">Don`t Repeat Yourself</span></strong><span data-type="color" style="color:#F5222D"> </span> is our principle.

> NOTE: The organization mode of Table-Driven is recommended by community. Click here to see more [details](https://github.com/golang/go/wiki/TableDrivenTests).

## 2.2 Mock － Analoging external dependences

Dependence problems are common in testing such as PouchContainer client needs HTTP server. But it is too heavy for units, and this belongs to integration testing. Therefore, how to complete this part of unit testing?

In golang world, the implementation of interface belongs to [Duck Type](https://en.wikipedia.org/wiki/Duck_typing). A certain interface is able to have a variety of implementations, as long as it conforms to the interface definition. If the external dependencies are constrained by interface, dependency behaviors are simulated in the unit testing. The following part introduces two common test situations.

### 2.2.1 RoundTripper
We use PouchContainer client test for example again, it uses [http.Client](https://golang.org/pkg/net/http/#Client), which uses interface called [RoundTripper](https://golang.org/pkg/net/http/#RoundTripper) to execute a HTTP request. It allows developers to custom logic of sending a HTTP request, which is the reason why golang are able to perfectly support HTTP 2 protocol upon original basement.

```plain
http.Client -> http.RoundTripper [http.DefaultTransport]
```

For PouchContainer client, testing mainly focuses on whether input destination address is correct, whether input query is reasonable and whether it can return correct results. Thus, before testing, developers need to be prepared for achievement of RounRripper. It is not responsible for the actual functional logic, just for judging whether the input is reasonable instead.
As you can see in the following codes, PouchContainer `newMockClient` could receive customized request logic. In the mock deleting instance, developer judged if the task address and HTTP Method is DELETE or not in customized logic, which enables function without running HTTP Server.

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
For internal packages' dependencies, such as PouchContainer Image API Bridge relies on PouchContainer Daemon ImageManager. But the dependencies in it defined by interface. If we want to test the logic of Image Bridge, we don't need to run containerd. You can implement the related Daemon ImageManager like RoundTripper.

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
ImageManager and RoundTripper are modeled in the same way except for the number of functions defined by the interface. In general, developers can manually define a structure that uses methods as fields, as shown in the next code.

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

When the interface is large and complex, the manual method will impose a test burden on the developers, so the community provides automatically generated tools, such as [mockery](https://github.com/vektra/mockery), to ease the burden on the developers.

## 2.3 Additionlly
Sometimes it relies on third-party services, take the PouchContainer client as a typical case.The above describes that Duck Type can complete the test of this case. In addition, we can also complete the request processing by registering the http.Handler and starting mockHTTPServer. It's a relatively harsh way of testing, which is recommended to be used only when the Duck Type test is unsuitable, or be put into the integration test.

> NOTE: Someone in the golang community modified the binary code to accompalish [monkeypatch](https://github.com/bouk/monkey), which is not recommended. Developers are recommended to design and write testable code.

### 2.4 Summary

PouchContainer integrates unit test cases into the code review phase, and reviewers can view the running of test cases at any time.

# 3.  <span data-type="color" style="color:rgb(34, 34, 34)"><span data-type="background" style="background-color:rgb(255, 255, 255)">Conclusion</span></span>

In the code review phase, code style checking, unit testing, and integration testing should be runned through continuous integration to help reviewers make accurate decisions. Currently, PouchContainer performs code style checks, testing and other operaions primarily through TravisCI/CircleCI and [pouchrobot](https://github.com/pouchcontainer/pouchrobot).
