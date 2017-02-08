# Validation
# Validation-유효성검사

- [Introduction](#introduction)
- [소개](#introduction)
- [Validation Quickstart](#validation-quickstart)
- [Validation Quickstart](#validation-quickstart)
    - [Defining The Routes](#quick-defining-the-routes)
    - [라우트 정의하기](#quick-defining-the-routes)
    - [Creating The Controller](#quick-creating-the-controller)
    - [컨트롤러 생성하기](#quick-creating-the-controller)
    - [Writing The Validation Logic](#quick-writing-the-validation-logic)
    - [Validation 로직 작성하기](#quick-writing-the-validation-logic)
    - [Displaying The Validation Errors](#quick-displaying-the-validation-errors)
    - [Validation 에러 표시하기](#quick-displaying-the-validation-errors)
    - [AJAX Requests & Validation](#quick-ajax-requests-and-validation)
    - [AJAX Requests & Validation](#quick-ajax-requests-and-validation)
    - [Validating Arrays](#validating-arrays)
    - [배열 유효성 검사](#validating-arrays)
- [Other Validation Approaches](#other-validation-approaches)
- [다른 Validation 방법](#other-validation-approaches)
    - [Manually Creating Validators](#manually-creating-validators)
    - [수동으로 Validator 생성하기](#manually-creating-validators)
    - [Form Request Validation](#form-request-validation)
    - [Form Request Validation](#form-request-validation)
- [Working With Error Messages](#working-with-error-messages)
- [에러 메시지 사용하기](#working-with-error-messages)
    - [Custom Error Messages](#custom-error-messages)
    - [사용자 지정(커스텀) 에러 메세지](#custom-error-messages)
- [Available Validation Rules](#available-validation-rules)
- [사용가능한 유효성 검사 규칙들](#available-validation-rules)
- [Conditionally Adding Rules](#conditionally-adding-rules)
- [조건부로 유효성 검사 규칙 추가하기](#conditionally-adding-rules)
- [Custom Validation Rules](#custom-validation-rules)
- [사용자 정의 유효성 검사 규칙 사용하기](#custom-validation-rules)

<a name="introduction"></a>
## Introduction
## 소개

Laravel provides several different approaches to validate your application's incoming data. By default, Laravel's base controller class uses a `ValidatesRequests` trait which provides a convenient method to validate incoming HTTP request with a variety of powerful validation rules.

라라벨은 어플리케이션에 유입되는 데이터의 유효성을 검사하기 위한 다양한 방법을 제공합니다. 기본적으로, 라라벨의 베이스 컨트롤러 클래스는 다양하고 강력한 유효성 검사 규칙을 적용하여 편리하게 HTTP 요청을 승인하는 메소드를 제공하는 `ValidatesRequests` 트레이트-trait을 사용하고 있습니다.

<a name="validation-quickstart"></a>
## Validation Quickstart
## Validation 퀵스타트

To learn about Laravel's powerful validation features, let's look at a complete example of validating a form and displaying the error messages back to the user.

라라벨의 강력한 유효성 검사 기능에 대해 알아보기 위해서, form을 확인한 뒤 사용자에게 에러 메세지를 보여주는 예제를 살펴보도록 하겠습니다. 

<a name="quick-defining-the-routes"></a>
### Defining The Routes
### 라우트 정의하기

First, let's assume we have the following routes defined in our `app/Http/routes.php` file:

우선 다음의 라우트들이 `app/Http/routes.php` 파일에 정의되어 있다고 가정해 보겠습니다:

    // Display a form to create a blog post...
    Route::get('post/create', 'PostController@create');

    // Store a new blog post...
    Route::post('post', 'PostController@store');

Of course, the `GET` route will display a form for the user to create a new blog post, while the `POST` route will store the new blog post in the database.

`GET` 라우트는 사용자가 새로운 블로그 포스트를 생성하기 위한 form을 나타낼 것이고, `POST` 라우트는 데이터베이스에 새로운 블로그 포스트를 저장할 것입니다. 

<a name="quick-creating-the-controller"></a>
### Creating The Controller
### 컨트롤러 생성하기

Next, let's take a look at a simple controller that handles these routes. We'll leave the `store` method empty for now:

다음으로, 이 라우트들을 다루는 간단한 컨트롤러를 살펴보겠습니다. 지금은 `store` 메소드를 비어있는 채로 둘 것입니다: 

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class PostController extends Controller
    {
        /**
         * Show the form the create a new blog post.
         *
         * @return Response
         */
        public function create()
        {
            return view('post.create');
        }

        /**
         * Store a new blog post.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // Validate and store the blog post...
        }
    }

<a name="quick-writing-the-validation-logic"></a>
### Writing The Validation Logic
### 유효성 검사 로직 작성하기

Now we are ready to fill in our `store` method with the logic to validate the new blog post. If you examine your application's base controller (`App\Http\Controllers\Controller`) class, you will see that the class uses a `ValidatesRequests` trait. This trait provides a convenient `validate` method in all of your controllers.

이제 새로운 블로그 포스트에 대해 유효성을 검사하는 로직을 `store` 메소드에 채워넣을 준비가 되었습니다. 어플리케이션의 베이스 컨트롤러(`App\Http\Controllers\Controller`) 클래스를 살펴보면 클래스가 `ValidatesRequests` 트레이트-trait을 사용한다는 것을 알 수 있습니다. 이 트레이트-trait은 모든 컨트롤러에 편리하게 사용할 수 있는 `validate` 메소드를 제공합니다. 

The `validate` method accepts an incoming HTTP request and a set of validation rules. If the validation rules pass, your code will keep executing normally; however, if validation fails, an exception will be thrown and the proper error response will automatically be sent back to the user. In the case of a traditional HTTP request, a redirect response will be generated, while a JSON response will be sent for AJAX requests.

`validate` 메소드는 HTTP 요청의 유입과 유효성 검사 룰의 집합을 전달 받습니다. 유효성 검사 룰들을 통과하게되면 코드는 계속해서 정상적으로 실행될 것입니다. 하지만 유효성 검사를 통과하지 못할 경우, 예외-exception가 던져지고 적절한 오류 응답이 사용자에게 자동으로 보내질 것입니다. 전통적인 HTTP 요청의 경우, 리다이렉트 응답이 생성될 것이며 AJAS 요청에는 JSON 응답이 보내질 것입니다.  

To get a better understanding of the `validate` method, let's jump back into the `store` method:

`validate` 메소드에 대해 더 잘 이해하기 위해, 다시 `store` 메소드로 돌아가 보겠습니다: 

    /**
     * Store a new blog post.
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        $this->validate($request, [
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
        ]);

        // The blog post is valid, store in database...
    }

As you can see, we simply pass the incoming HTTP request and desired validation rules into the `validate` method. Again, if the validation fails, the proper response will automatically be generated. If the validation passes, our controller will continue executing normally.

위에서 볼 수 있듯이, 간단하게 유입되는 HTTP 요청과 유효성 검사 룰들을 `validate` 메소드로 전달하면 됩니다. 이 때에도 유효 확인이 실패하면 적절한 응답이 생성될 것입니다. 유효성 검사를 통과하면 컨트롤러는 계속해서 정상적으로 수행합니다.

#### Stopping On First Validation Failure
#### 첫번째 유효성 검사가 실패하면 중지하기

Sometimes you may wish to stop running validation rules on an attribute after the first validation failure. To do so, assign the `bail` rule to the attribute:

경우에 따라서 첫번째 유효성 감사가 실패하면 속성값에 대한 검사를 중단하기를 원할수도 있습니다. 이러한 경우 `bail` 규칙을 지정하면 됩니다.

    $this->validate($request, [
        'title' => 'bail|required|unique:posts|max:255',
        'body' => 'required',
    ]);

In this example, if the `required` rule on the `title` attribute fails, the `unique` rule will not be checked. Rules will be validated in the order they are assigned.

이 예제에서는, `required` 규칙으로 지정된(필수값) `title` 속성의 유효성 검사가 실패하면 `unique` 규칙은 확인하지 않습니다. 유효성 검사 규칙은 선언된 순서대로 검사될 것입니다. 

#### A Note On Nested Attributes
#### 중첩된 속성에 대한 유의사항

If your HTTP request contains "nested" parameters, you may specify them in your validation rules using "dot" syntax:

HTTP 요청이 "중첩된" 파라미터를 가지고 있다면 ".(점)" 문법을 사용하여 유효성 확인 규칙을 지정할 수 있습니다:

    $this->validate($request, [
        'title' => 'required|unique:posts|max:255',
        'author.name' => 'required',
        'author.description' => 'required',
    ]);

<a name="quick-displaying-the-validation-errors"></a>
### Displaying The Validation Errors
### 유효성 검사 에러를 보여주기

So, what if the incoming request parameters do not pass the given validation rules? As mentioned previously, Laravel will automatically redirect the user back to their previous location. In addition, all of the validation errors will automatically be [flashed to the session](/docs/{{version}}/session#flash-data).

그럼 입력되는 요청-request의 파라미터들이 유효성 검사를 통과하지 못하는 경우에는 어떻게 될까요? 이 경우 라라벨은 앞서 언급한대로, 사용자를 자동으로 이전의 위치로  리다이렉트합니다. 또한 모든 유효성 확인 에러는 자동으로 [세션에 임시 저장](/docs/{{version}}/session#flash-data)될 것입니다. 

Again, notice that we did not have to explicitly bind the error messages to the view in our `GET` route. This is because Laravel will check for errors in the session data, and automatically bind them to the view if they are available. The `$errors` variable will be an instance of `Illuminate\Support\MessageBag`. For more information on working with this object, [check out its documentation](#working-with-error-messages).

이번에도 `GET` 라우트에서 에러 메세지를 뷰와 명시적으로 연결하지 않아도 됩니다. 왜냐하면, 라라벨이 세션 데이터에서 에러가 있는지 확인하고, 에러가 있다면 뷰에 자동으로 연결해 주기 때문입니다. 따라서 여러분은 `$errors` 변수가 항상 정의되어 있으며 사용 가능하다고 마음 편하게 가정할 수 있습니다. `$errors` 변수는 `Illuminate\Support\MessageBag`의 인스턴스일 것입니다. 이 객체를 다루는 법에 대해 더 알아보고 싶다면 [문서을 확인해보시기 바랍니다](#working-with-error-messages).

> **Note:** The `$errors` variable is bound to the view by the `Illuminate\View\Middleware\ShareErrorsFromSession` middleware, which is provided by the `web` middleware group. **When this middleware is applied an `$errors` variable will always be available in your views**, allowing you to conveniently assume the `$errors` variable is always defined and can be safely used.

> **참고:** `$errors` 변수는 `web` 미들웨어 그룹에 의해서 제공되는 `Illuminate\View\Middleware\ShareErrorsFromSession` 미들웨어에 의해서 뷰와 연결됩니다. **이 미들웨어가 지정되었을 때 `$errors` 변수는 뷰 안에서 항상 사용가능 할 것이므로**, 편하게 생각하면 `$errors` 변수는 항상 선언되어 있다고 생각하고 안정하게 사용할 수 있습니다. 

So, in our example, the user will be redirected to our controller's `create` method when validation fails, allowing us to display the error messages in the view:

따라서, 이 예제에서, 유효성 검사를 통과하지 못할 경우 사용자는 컨트롤러의 `create` 메소드로 리다이렉트 될것이고, 뷰에서는 에러 메세지가 표시됩니다: 

    <!-- /resources/views/post/create.blade.php -->

    <h1>Create Post</h1>

    @if (count($errors) > 0)
        <div class="alert alert-danger">
            <ul>
                @foreach ($errors->all() as $error)
                    <li>{{ $error }}</li>
                @endforeach
            </ul>
        </div>
    @endif

    <!-- Create Post Form -->

<a name="quick-customizing-the-flashed-error-format"></a>
#### Customizing The Flashed Error Format
#### 임시저장된 에러의 포맷을 임의로 지정하기

If you wish to customize the format of the validation errors that are flashed to the session when validation fails, override the `formatValidationErrors` on your base controller. Don't forget to import the `Illuminate\Contracts\Validation\Validator` class at the top of the file:

만약 유효성 검사가 실패했을 때 세션에 저장되는 에러의 형식을 커스터마이징하고 싶다면, 베이스 컨트롤러 클래스의 `formatErrors` 메소드를 오버라이딩하면 됩니다. 이때, 파일 상단에서 `Illuminate\Validation\Validator`를 임포트하는 것을 잊지 마십시오:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Foundation\Bus\DispatchesJobs;
    use Illuminate\Contracts\Validation\Validator;
    use Illuminate\Routing\Controller as BaseController;
    use Illuminate\Foundation\Validation\ValidatesRequests;

    abstract class Controller extends BaseController
    {
        use DispatchesJobs, ValidatesRequests;

        /**
         * {@inheritdoc}
         */
        protected function formatValidationErrors(Validator $validator)
        {
            return $validator->errors()->all();
        }
    }

<a name="quick-ajax-requests-and-validation"></a>
### AJAX Requests & Validation
### AJAX 요청과 유효성 검사

In this example, we used a traditional form to send data to the application. However, many applications use AJAX requests. When using the `validate` method during an AJAX request, Laravel will not generate a redirect response. Instead, Laravel generates a JSON response containing all of the validation errors. This JSON response will be sent with a 422 HTTP status code.

이 예제에서는, 어플리케이션에 전통적인 form을 이용하여 데이터를 보냈습니다. 하지만 많은 어플리케이션이 AJAX 요청을 사용합니다. AJAX reqeust 중에서 `validate` 메소드를 사용한다면 라라벨은 리다이렉트 응답을 생성하지 않을 것입니다. 대신 라라벨은 유효성 검사의 모든 실패 에러들을 포함하는 JSON 응답을 생성할 것입니다. 이 JSON 응답은 422 HTTP 상태 코드와 함께 보내질 것입니다.

<a name="validating-arrays"></a>
### Validating Arrays
### 배열 유효성 검사

Validating array form input fields doesn't have to be a pain. For example, to validate that each e-mail in a given array input field is unique, you may do the following:

배열 form 필드에 대해서 유효성 검사를 하는 것은 어렵지 않습니다. 예를 들어 주어진 배열 입력필드의 각각의 이메일이 고유한지 검증하려면 다음처럼 하면 됩니다: 

    $validator = Validator::make($request->all(), [
        'person.*.email' => 'email|unique:users',
        'person.*.first_name' => 'required_with:person.*.last_name',
    ]);

Likewise, you may use the `*` character when specifying your validation messages in your language files, making it a breeze to use a single validation message for array based fields:

마찬가지로, 언어 파일에서 유효성 검사 메세지를 지정하는데, `*` 문자를 사용할 수 있으며, 이는 배열기반의 필드에 대한 유효성 검사 메세지를 손쉽게 구성할 수 있게 만듭니다:

    'custom' => [
        'person.*.email' => [
            'unique' => 'Each person must have a unique e-mail address',
        ]
    ],

<a name="other-validation-approaches"></a>
## Other Validation Approaches
## 다른 유효성 검사 방법

<a name="manually-creating-validators"></a>
### Manually Creating Validators
### 수동으로 Validator 생성하기

If you do not want to use the `ValidatesRequests` trait's `validate` method, you may create a validator instance manually using the `Validator` [facade](/docs/{{version}}/facades). The `make` method on the facade generates a new validator instance:

`ValidatesRequests` 트레이트-trait의 `validate` 메소드를 사용하고 싶지 않다면 `Validator` [파사드](/docs/{{version}}/facades)를 사용하여 validator 인스턴스를 수동으로 생성할 수 있습니다. 파사드에 `make` 메소드를 사용하면 새로운 validator 인스턴스가 생성됩니다:

    <?php

    namespace App\Http\Controllers;

    use Validator;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class PostController extends Controller
    {
        /**
         * Store a new blog post.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            $validator = Validator::make($request->all(), [
                'title' => 'required|unique:posts|max:255',
                'body' => 'required',
            ]);

            if ($validator->fails()) {
                return redirect('post/create')
                            ->withErrors($validator)
                            ->withInput();
            }

            // Store the blog post...
        }
    }

The first argument passed to the `make` method is the data under validation. The second argument is the validation rules that should be applied to the data.

`make` 메소드로 전달되는 첫번째 인자는 유효성 검사를 받을 데이터입니다. 두번째 인자는 데이터에 적용되어야 하는 유효성 검사 규칙들입니다. 

After checking if the request failed to pass validation, you may use the `withErrors` method to flash the error messages to the session. When using this method, the `$errors` variable will automatically be shared with your views after redirection, allowing you to easily display them back to the user. The `withErrors` method accepts a validator, a `MessageBag`, or a PHP `array`.

request-요청이 유효성 검사를 통과하지 못한 것을 확인였다면 `withErrors` 메소드로 세션에 에러 메세지를 임시저장-flash 할 수 있습니다. 이 메소드를 사용하면 리다이렉트 후에 `$errors` 변수가 자동으로 뷰에서 공유되어 손쉽게 사용자에게 보여질 수 있습니다. `withErrors` 메소드는 validator, `MessageBag`, 혹은 PHP `array`를 전달 받습니다. 

#### Named Error Bags
#### 이름이 지정된 Error Bags

If you have multiple forms on a single page, you may wish to name the `MessageBag` of errors, allowing you to retrieve the error messages for a specific form. Simply pass a name as the second argument to `withErrors`:

한 페이지 안에서 여러개의 form을 가지고 있다면 에러들의 `MessageBag`에 이름을 붙여 지정한 form에 맞는 에러 메세지를 조회할 수 있도록 할 수 있습니다. 단순히 `withErrors`에 이름을 두번째 인자로 전달하면 됩니다:

    return redirect('register')
                ->withErrors($validator, 'login');

You may then access the named `MessageBag` instance from the `$errors` variable:

그러면 `$errors` 변수에서 지정된 `MessageBag` 인스턴스에 접근할 수 있습니다. 

    {{ $errors->login->first('email') }}

#### After Validation Hook
#### 유효성 검사 이후에 후킹하기

The validator also allows you to attach callbacks to be run after validation is completed. This allows you to easily perform further validation and even add more error messages to the message collection. To get started, use the `after` method on a validator instance:

유효성 검사가 완료된 후에 실행하고자 하는 콜백함수를 validator에 추가할 수 있습니다. 이를 통하면, 손쉽게 더 많은 유효성 검사를 실행할 수 있도록 하고, 에러 메시지 컬렉션에 에러 메시지를 더 추가할 수도 있습니다. 다음처럼 validator 인스턴스의 `after` 메소드를 사용하면 됩니다:

    $validator = Validator::make(...);

    $validator->after(function($validator) {
        if ($this->somethingElseIsInvalid()) {
            $validator->errors()->add('field', 'Something is wrong with this field!');
        }
    });

    if ($validator->fails()) {
        //
    }

<a name="form-request-validation"></a>
### Form Request Validation
### Form request-요청 유효성 검사

For more complex validation scenarios, you may wish to create a "form request". Form requests are custom request classes that contain validation logic. To create a form request class, use the `make:request` Artisan CLI command:

보다 복잡한 유효성 검사 시나리오를 고려해보면, "폼 요청(form request)" 클래스를 만들고자 할 수도 있습니다. form reqeust는 유효성검사 로직을 내장하고 있는 사용자 정의 request-요청 클래스입니다. 하나의 form request-요청 클래스를 만드려면, `make:request` 아티즌 명령을 사용하면 됩니다.

    php artisan make:request StoreBlogPostRequest

The generated class will be placed in the `app/Http/Requests` directory. Let's add a few validation rules to the `rules` method:

생성된 request-요청 클래스는 `app/Http/Requests` 디렉토리에 위치합니다. 유효성 검사 규칙 몇개를 `rules` 메소드에 추가해보겠습니다:

    /**
     * Get the validation rules that apply to the request.
     *
     * @return array
     */
    public function rules()
    {
        return [
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
        ];
    }

So, how are the validation rules evaluated? All you need to do is type-hint the request on your controller method. The incoming form request is validated before the controller method is called, meaning you do not need to clutter your controller with any validation logic:

그럼 어떻게 이 유효성 검사 규칙이 실행되도록 할 수 있을까요? 필요한 것은 여러분의 컨트롤러 메소드에 해당 request-요청 클래스의 타입힌트를 넣어주는 것입니다. form request-요청은 컨트롤러 메소드가 호출되기 전에 미리 유효성 검사를 거칩니다. 이는 유효성 검사 로직을 위해 여러분의 컨트롤러를 지저분하게 할 필요가 없다는 것을 의미합니다.

    /**
     * Store the incoming blog post.
     *
     * @param  StoreBlogPostRequest  $request
     * @return Response
     */
    public function store(StoreBlogPostRequest $request)
    {
        // The incoming request is valid...
    }

If validation fails, a redirect response will be generated to send the user back to their previous location. The errors will also be flashed to the session so they are available for display. If the request was an AJAX request, a HTTP response with a 422 status code will be returned to the user including a JSON representation of the validation errors.

만약 유효성 검사가 실패하면, 이전 페이지로의 리다이렉트 응답이 생성되어 사용자에게 보내집니다. 에러들 또한 화면에 출력될 수 있도록 세션에 기록될 것입니다. 만약 요청이 AJAX 요청이면, 422 상태 코드의 HTTP 응답이 JSON 형식의 오류 메시지를 포함한 채 사용자에게 반환됩니다.

#### Authorizing Form Requests
#### Form Requests authorizing-승인

The form request class also contains an `authorize` method. Within this method, you may check if the authenticated user actually has the authority to update a given resource. For example, if a user is attempting to update a blog post comment, do they actually own that comment? For example:

또한 form request-요청 클래스는 `authorize` 메소드를 가지고 있습니다. 이 메소드 안에서, 여러분은 사용자가 실제로 주어진 리소스를 업데이트할 권한을 가졌는지 확인 할 수 있습니다. 다음은 만약 사용자가 블로그 포스트의 댓글을 업데이트하려고 시도할 때, 실제로 사용자가 그 댓글의 소유자인가를 확인하는 예제입니다:

    /**
     * Determine if the user is authorized to make this request.
     *
     * @return bool
     */
    public function authorize()
    {
        $commentId = $this->route('comment');

        return Comment::where('id', $commentId)
                      ->where('user_id', Auth::id())->exists();
    }

Note the call to the `route` method in the example above. This method grants you access to the URI parameters defined on the route being called, such as the `{comment}` parameter in the example below:

이 예제에서 `route` 메소드를 호출하는 것에 주목하십시오. 예제에서 이 메소드는 `{comment}` 파라미터와 같이 파라미터가 정의된 라우트의 URI 에 여러분이 엑세스 접근할 수 있는지 권한을 확인합니다:

    Route::post('comment/{comment}');

If the `authorize` method returns `false`, a HTTP response with a 403 status code will automatically be returned and your controller method will not execute.

만약 `authorize` 메소드가 `false`를 리턴하면, 403 HTTP 응답이 자동적으로 반환되고 컨트롤러 메소드는 실행되지 않을 것입니다.

If you plan to have authorization logic in another part of your application, simply return `true` from the `authorize` method:

여러분이 어플리케이션의 다른 부분에 있는 인증로직을 사용할 계획이라면, 그냥 `authorize` 메소드에서 `true`를 리턴하면 됩니다.

    /**
     * Determine if the user is authorized to make this request.
     *
     * @return bool
     */
    public function authorize()
    {
        return true;
    }

#### Customizing The Flashed Error Format
#### 임시저장된 에러의 포맷을 임의로 지정하기

If you wish to customize the format of the validation errors that are flashed to the session when validation fails, override the `formatErrors` on your base request (`App\Http\Requests\Request`). Don't forget to import the `Illuminate\Contracts\Validation\Validator` class at the top of the file:

만약 유효성 검사가 실패했을 때 세션에 저장되는 에러의 형식을 커스터마이징하고 싶다면, 베이스 request-요청 클래스(`App\Http\Requests\Request`)의 `formatErrors` 메소드를 오버라이딩하면 됩니다. 이때, 파일 상단에서 `Illuminate\Validation\Validator`를 임포트하는 것을 잊지 마십시오:

    /**
     * {@inheritdoc}
     */
    protected function formatErrors(Validator $validator)
    {
        return $validator->errors()->all();
    }

#### Customizing The Error Messages
#### 에러 메세지를 사용자 정의하기(커스터마이징하기)

You may customize the error messages used by the form request by overriding the `messages` method. This method should return an array of attribute / rule pairs and their corresponding error messages:

`messages` 메소드를 대체하면 form request-요청이 사용하는 에러 메세지를 커스터마이즈할 수 있습니다. 이 메소드는 속성 / 규칙의 쌍으로된 배열과 그에 상응하는 오류 메세지를 반환합니다: 

    /**
     * Get the error messages for the defined validation rules.
     *
     * @return array
     */
    public function messages()
    {
        return [
            'title.required' => 'A title is required',
            'body.required'  => 'A message is required',
        ];
    }

<a name="working-with-error-messages"></a>
## Working With Error Messages
## 에러 메시지 사용하기

After calling the `errors` method on a `Validator` instance, you will receive an `Illuminate\Support\MessageBag` instance, which has a variety of convenient methods for working with error messages.

`Validator` 인스턴스의 `messages` 메소드를 호출하면, 에러 메시지를 편하게 사용할 수 있는 다양한 메소드를 가진 `MessageBag` 인스턴스를 받을 수 있습니다.

#### Retrieving The First Error Message For A Field
#### 하나의 필드에 대한 첫번째 에러 메시지 조회하기

To retrieve the first error message for a given field, use the `first` method:

특정 필드에 대한 첫번째 에러 메세지를 조회하려면 `first` 메소드를 사용하면 됩니다: 

    $messages = $validator->errors();

    echo $messages->first('email');

#### Retrieving All Error Messages For A Field
#### 하나의 필드에 대한 모든 에러 메세지 조회하기

If you wish to simply retrieve an array of all of the messages for a given field, use the `get` method:

간단하게 하나의 필드에 대한 모든 에러 메세지를 조회하고 싶다면 `get` 메소드를 사용하면 됩니다:

    foreach ($messages->get('email') as $message) {
        //
    }

#### Retrieving All Error Messages For All Fields
#### 모든 필드에 대한 모든 에러 메세지 조회하기

To retrieve an array of all messages for all fields, use the `all` method:

모든 필드에 대한 모든 에러 메세지를 조회하기 위해서는 `all` 메소들 사용하면 됩니다: 

    foreach ($messages->all() as $message) {
        //
    }

#### Determining If Messages Exist For A Field
#### 하나의 필드에 대하여 에러 메시지가 존재하는지 검사하기

    if ($messages->has('email')) {
        //
    }

#### Retrieving An Error Message With A Format 
#### 특정 형식으로 에러 메시지 획득하기

    echo $messages->first('email', '<p>:message</p>');

#### Retrieving All Error Messages With A Format 
#### 특정 형식으로 모든 에러 메시지 획득하기

    foreach ($messages->all('<li>:message</li>') as $message) {
        //
    }

<a name="custom-error-messages"></a>
### Custom Error Messages
### 사용자 지정(커스텀) 에러 메세지

If needed, you may use custom error messages for validation instead of the defaults. There are several ways to specify custom messages. First, you may pass the custom messages as the third argument to the `Validator::make` method:

필요하다면 기본적인 에얼 메세지 대신에 커스텀 에러 메세지를 유효성 검사에 사용할 수 있습니다. 커스텀 메세지를 지정하는 데에는 여러가지 방법이 있습니다. 먼저 `Validator::make` 메소드에 커스텀 메세지를 세번째 인자로 전달할 수 있습니다: 

    $messages = [
        'required' => 'The :attribute field is required.',
    ];

    $validator = Validator::make($input, $rules, $messages);

In this example, the `:attribute` place-holder will be replaced by the actual name of the field under validation. You may also utilize other place-holders in validation messages. For example:

다음의 예에서 `:attribute` 플레이스 홀더는 유효성 검사를 받는 필드의 실제 이름으로 대체됩니다. 유효성 검사 메세지에서 다른 플레이스 홀더들 또한 활용할 수 있습니다. 예를 들어: 

    $messages = [
        'same'    => 'The :attribute and :other must match.',
        'size'    => 'The :attribute must be exactly :size.',
        'between' => 'The :attribute must be between :min - :max.',
        'in'      => 'The :attribute must be one of the following types: :values',
    ];

#### Specifying A Custom Message For A Given Attribute
#### 주어진 속성에 대해 커스텀 메세지 지정하기 

Sometimes you may wish to specify a custom error messages only for a specific field. You may do so using "dot" notation. Specify the attribute's name first, followed by the rule:

종종 하나의 특정 필드에 대해서만 커스텀 오류 메세지를 지정해야 하는 경우가 있습니다. 이것은 ".(점)" 표기법을 통해서 할 수 있습니다. 속성의 이름을 먼저 지정하고, 규칙을 명시하면됩니다: 

    $messages = [
        'email.required' => 'We need to know your e-mail address!',
    ];

<a name="localization"></a>
#### Specifying Custom Messages In Language Files
#### 언어 파일에 커스텀 메세지 지정하기 

In many cases, you may wish to specify your attribute specific custom messages in a language file instead of passing them directly to the `Validator`. To do so, add your messages to `custom` array in the `resources/lang/xx/validation.php` language file.

많은 경우에서, `Validator`에 직접 메세지를 전달하는 대신, 언어 파일에 속성에 따른 커스텀 메세지를 지정하기 원할 수 있습니다. 이렇게 하기 위해서는 `resources/lang/xx/validation.php` 언어 파일의 `custom` 배열에 메제지를 추가하면 됩니다.

    'custom' => [
        'email' => [
            'required' => 'We need to know your e-mail address!',
        ],
    ],

<a name="available-validation-rules"></a>
## Available Validation Rules
## 사용가능한 유효성 검사 규칙

Below is a list of all available validation rules and their function: 

다음은 모든 유효성 검사 규칙과 그들의 함수 목록입니다.

<style>
    .collection-method-list > p {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        column-gap: 2em; -moz-column-gap: 2em; -webkit-column-gap: 2em;
    }

    .collection-method-list a {
        display: block;
    }
</style>

[Accepted](#rule-accepted)
[Active URL](#rule-active-url)
[After (Date)](#rule-after)
[Alpha](#rule-alpha)
[Alpha Dash](#rule-alpha-dash)
[Alpha Numeric](#rule-alpha-num)
[Array](#rule-array)
[Before (Date)](#rule-before)
[Between](#rule-between)
[Boolean](#rule-boolean)
[Confirmed](#rule-confirmed)
[Date](#rule-date)
[Date Format](#rule-date-format)
[Different](#rule-different)
[Digits](#rule-digits)
[Digits Between](#rule-digits-between)
[Dimensions (Image Files)](#rule-dimensions)
[Distinct](#rule-distinct)
[E-Mail](#rule-email)
[Exists (Database)](#rule-exists)
[File](#rule-file)
[Filled](#rule-filled)
[Image (File)](#rule-image)
[In](#rule-in)
[In Array](#rule-in-array)
[Integer](#rule-integer)
[IP Address](#rule-ip)
[JSON](#rule-json)
[Max](#rule-max)
[MIME Types](#rule-mimetypes)
[MIME Type By File Extension](#rule-mimes)
[Min](#rule-min)
[Not In](#rule-not-in)
[Numeric](#rule-numeric)
[Present](#rule-present)
[Regular Expression](#rule-regex)
[Required](#rule-required)
[Required If](#rule-required-if)
[Required Unless](#rule-required-unless)
[Required With](#rule-required-with)
[Required With All](#rule-required-with-all)
[Required Without](#rule-required-without)
[Required Without All](#rule-required-without-all)
[Same](#rule-same)
[Size](#rule-size)
[String](#rule-string)
[Timezone](#rule-timezone)
[Unique (Database)](#rule-unique)
[URL](#rule-url)

<a name="rule-accepted"></a>
#### accepted

The field under validation must be _yes_, _on_, _1_, or _true_. This is useful for validating "Terms of Service" acceptance.

필드의 값이 _yes_, _on_, _1_, 또는 _true_이어야 합니다. 이 것은 "이용약관" 동의와 같은 필드의 검사에 유용합니다.

<a name="rule-active-url"></a>
#### active_url

The field under validation must be a valid URL according to the `checkdnsrr` PHP function.

필드의 값이 PHP 함수 `checkdnsrr`에 기반하여 올바른 URL이어야 합니다.

<a name="rule-after"></a>
#### after:_date_

The field under validation must be a value after a given date. The dates will be passed into the `strtotime` PHP function:

필드의 값이 주어진 날짜 이후여야 합니다. 이때 날짜는 `strtotime` PHP 함수를 통해 생성된 값입니다.

    'start_date' => 'required|date|after:tomorrow'

Instead of passing a date string to be evaluated by `strtotime`, you may specify another field to compare against the date:

`strtotime`에 의해 계산될 날짜 문자열을 전달하는 대신 날짜와 비교할 다른 필드를 명시할 수 있습니다: 

    'finish_date' => 'required|date|after:start_date'

<a name="rule-alpha"></a>
#### alpha

The field under validation must be entirely alphabetic characters.

필드의 값이 완벽하게 (숫자나 기호가 아닌) 알파벳[자음과 모음] 문자로 이루어져야 합니다.

(역자주: 영문 알파벳만을 의미하지 않고, 숫자나 기호가 아닌경우에 해당하여, 한글도 허용합니다.)

<a name="rule-alpha-dash"></a>
#### alpha_dash

The field under validation may have alpha-numeric characters, as well as dashes and underscores.

필드의 값이 (숫자나 기호가 아닌) 알파벳[자음과 모음] 문자 및 숫자와 dash(-), underscore(_)로 이루어져야 합니다.

<a name="rule-alpha-num"></a>
#### alpha_num

The field under validation must be entirely alpha-numeric characters.

필드의 값이 완벽하게 (숫자나 기호가 아닌) 알파벳[자음과 모음] 문자 및 숫자로 이루어져야 합니다.

<a name="rule-array"></a>
#### array

The field under validation must be a PHP `array`.

필드의 값이 반드시 PHP 배열 형태이어야 합니다.

<a name="rule-before"></a>
#### before:_date_

The field under validation must be a value preceding the given date. The dates will be passed into the PHP `strtotime` function.

필드의 값이 반드시 주어진 날짜보다 앞서야 합니다. 날짜는 `strtotime` PHP 함수를 통해 비교됩니다.

<a name="rule-between"></a>
#### between:_min_,_max_

The field under validation must have a size between the given _min_ and _max_. Strings, numerics, and files are evaluated in the same fashion as the [`size`](#rule-size) rule.

필드의 값이, 주어진 _min_ 과 _max_의 사이의 값이어야 합니다. 문자열, 숫자, 그리고 파일이 [`size`](#rule-size) 룰에 의해 같은 방식으로 평가될 수 있습니다.

<a name="rule-boolean"></a>
#### boolean

The field under validation must be able to be cast as a boolean. Accepted input are `true`, `false`, `1`, `0`, `"1"`, and `"0"`.

필드의 값이 반드시 boolean으로 캐스팅될 수 있어야 합니다. 허용되는 값은 `true`, `false`, `1`, `0`, `"1"`, `"0"` 입니다.

<a name="rule-confirmed"></a>
#### confirmed

The field under validation must have a matching field of `foo_confirmation`. For example, if the field under validation is `password`, a matching `password_confirmation` field must be present in the input. 

필드의 값이 `foo_confirmation`의 매칭되는 필드를 가져야 합니다. 예를 들어 만약 필드가 `password`라면, `password_confirmation`라는 필드가 입력값 중에 있어야 합니다.

<a name="rule-date"></a>
#### date

The field under validation must be a valid date according to the `strtotime` PHP function.

필드의 값이 `strtotime` PHP 함수에서 인식할 수 있는 올바른 날짜여야 합니다.

<a name="rule-date-format"></a>
#### date_format:_format_

The field under validation must match the given _format_. The format will be evaluated using the PHP `date_parse_from_format` function. You should use **either** `date` or `date_format` when validating a field, not both.

필드의 값이 반드시 주어진 _format_과 일지해야 합니다. 주어진 포맷은 `date_parse_from_format` PHP 함수에 의해서 연산될 것입니다. 필드의 유효성을 검사할 때에는 `date`와 `date_format` 중 **하나만** 사용해야 합니다.

<a name="rule-different"></a>
#### different:_field_

The field under validation must have a different value than _field_.

필드의 값이 주어진 _field_의 값과 달라야 합니다.

<a name="rule-digits"></a>
#### digits:_value_

The field under validation must be _numeric_ and must have an exact length of _value_. 

필드의 값이 반드시 _숫자_여야 하고, 길이가 _value_이어야 합니다.

<a name="rule-digits-between"></a>
#### digits_between:_min_,_max_

The field under validation must have a length between the given _min_ and _max_.

필드의 값이 주어진 _min_과 _max_ 사이의 길이를 가져야 합니다.

<a name="rule-dimensions"></a>
#### dimensions

The file under validation must be an image meeting the dimension constraints as specified by the rule's parameters:

필드의 값이 룰에 지정된 파라미터들을 만족하는 이미지이어야 합니다.

    'avatar' => 'dimensions:min_width=100,min_height=200'

Available constraints are: _min\_width_, _max\_width_, _min\_height_, _max\_height_, _width_, _height_, _ratio_.

<a name="rule-distinct"></a>
#### distinct

When working with arrays, the field under validation must not have any duplicate values.

배열에서 동작하며, 필드의 값이 배열안의 다른 값과 중복되지 않아야 합니다.

    'foo.*.id' => 'distinct'

<a name="rule-email"></a>
#### email

The field under validation must be formatted as an e-mail address. 

필드의 값이 이메일 주소 형식이어야 합니다.

<a name="rule-exists"></a>
#### exists:_table_,_column_

The field under validation must exist on a given database table.

필드의 값이 주어진 데이터베이스 테이블에 존재하는 값이어야 합니다.

#### Basic Usage Of Exists Rule 
#### exists 룰의 기본 사용법

    'state' => 'exists:states'

#### Specifying A Custom Column Name 
#### 특정 컬럼명 지정하기

    'state' => 'exists:states,abbreviation'

You may also specify more conditions that will be added as "where" clauses to the query:

쿼리문의 "where" 구문에 추가될 더 많은 조건을 지정할 수도 있습니다: 

    'email' => 'exists:staff,email,account_id,1'

These conditions may be negated using the `!` sign:

이러한 조건은 `!` 사인을 사용하여 부정할 수 있습니다. 

    'email' => 'exists:staff,email,role,!admin'

You may also pass `NULL` or `NOT_NULL` to the "where" clause:

"where" 구분에 `NULL` 혹은 `NOT_NULL`을 전달할 수도 있습니다: 

    'email' => 'exists:staff,email,deleted_at,NULL'

    'email' => 'exists:staff,email,deleted_at,NOT_NULL'

Occasionally, you may need to specify a specific database connection to be used for the `exists` query. You can accomplish this by prepending the connection name to the table name using "dot" syntax:

때때로, `exists` 쿼리에서 사용할 데이터베이스 커넥션을 지정하고자 할 수도 있습니다. 이경우 커넥션의 이름을 테이블 이름 앞에 "점" 문법을 사용하여 표기할 수도 있습니다.

    'email' => 'exists:connection.staff,email'

<a name="rule-file"></a>
#### file

The field under validation must be a successfully uploaded file.

필드의 값이 완전히 업로드된 파일이어야 합니다.

<a name="rule-filled"></a>
#### filled

The field under validation must not be empty when it is present.

필드가 존재하는 경우 값이 비어있으면 안됩니다.

<a name="rule-image"></a>
#### image

The file under validation must be an image (jpeg, png, bmp, gif, or svg) 

이미지 파일(jpeg, png, bmp, gif, svg)이어야 합니다.

<a name="rule-in"></a>
#### in:_foo_,_bar_,...

The field under validation must be included in the given list of values. 

필드의 값이 주어진 목록에 포함돼 있어야 합니다.

<a name="rule-in-array"></a>
#### in_array:_anotherfield_

The field under validation must exist in _anotherfield_'s values.

필드의 값이 주어진 다른 필드의 값안에 존재해야만 합니다.

<a name="rule-integer"></a>
#### integer

The field under validation must have an integer value. 

필드의 값이 정수여야 합니다.

<a name="rule-ip"></a>
#### ip

The field under validation must be an IP address.

필드의 값이 IP 주소여야 합니다.

<a name="rule-json"></a>
#### json

The field under validation must be a valid JSON string.

필드의 값이 유효한 JSON 문자열이어야 합니다. 

<a name="rule-max"></a>
#### max:_value_

The field under validation must be less than or equal to a maximum _value_. Strings, numerics, and files are evaluated in the same fashion as the [`size`](#rule-size) rule.

필드의 값이 반드시 _value_보다 작거나 같아야 합니다. 문자열, 숫자, 그리고 파일이 [`size`](#rule-size) 룰에 의해 같은 방식으로 평가될 수 있습니다.

<a name="rule-mimetypes"></a>
#### mimetypes:_text/plain_,...

The file under validation must match one of the given MIME types:

파일이 주어진 MIME 타입들 중 하나와 일치해야만 합니다.

    'video' => 'mimetypes:video/avi,video/mpeg,video/quicktime'

To determine the MIME type of the uploaded file, the file's contents will be read and the framework will attempt to guess the MIME type, which may be different from the client provided MIME type.

업로드 파일의 MIME 타입을 지정하고자 한다면, 프레임 워크는 파일의 내용을 읽어 들여 MIME 타입을 추측하게 되며 클라이언트가 제공하는 MIME 타입과 달라질 수 있습니다.  

<a name="rule-mimes"></a>
#### mimes:_foo_,_bar_,...

The file under validation must have a MIME type corresponding to one of the listed extensions.

파일의 MIME 타입이 주어진 확장자 리스트 중에 하나와 일치해야 합니다.

#### Basic Usage Of MIME Rule
#### MIME 룰의 기본 사용법

    'photo' => 'mimes:jpeg,bmp,png'

Even though you only need to specify the extensions, this rule actually validates against the MIME type of the file by reading the file's contents and guessing its MIME type.

여러분은 확장자를 지정하기만 하면 되지만, 이 경우 파일의 컨텐트를 읽고 MIME 타입을 추정함으로써 이 파일의 MIME의 유효성을 검사합니다.

A full listing of MIME types and their corresponding extensions may be found at the following location: [http://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types](http://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types)

MIME 타입과 그에 상응하는 확장의 전체 목록은 다음의 위치에서 확인하실 수 있습니다: [http://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types](http://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types)

<a name="rule-min"></a>
#### min:_value_

The field under validation must have a minimum _value_. Strings, numerics, and files are evaluated in the same fashion as the [`size`](#rule-size) rule.

필드의 값이 반드시 _value_ 보다 크거나 같아야 합니다. 문자열, 숫자, 그리고 파일이 [`size`](#rule-size) 룰에 의해 같은 방식으로 평가될 수 있습니다.

<a name="rule-not-in"></a>
#### not_in:_foo_,_bar_,...

The field under validation must not be included in the given list of values. 

필드의 값이 주어진 목록에 존재하지 않아야 합니다.

<a name="rule-numeric"></a>
#### numeric

The field under validation must be numeric.

필드의 값이 숫자여야 합니다.

<a name="rule-present"></a>
#### present

The field under validation must be present in the input data but can be empty.

필드가 존재하고 있는지 확인하지만, 값이 비어있을 수 있습니다.

<a name="rule-regex"></a>
#### regex:_pattern_

The field under validation must match the given regular expression.

필드의 값이 주어진 정규식 표현과 일치해야 합니다.

**Note:** When using the `regex` pattern, it may be necessary to specify rules in an array instead of using pipe delimiters, especially if the regular expression contains a pipe character. 

**참고:** `regex` 패턴을 사용할 때, 특히 정규 표현식에 파이프 문자열이 있다면, 파이프 구분자를 사용하는 대신 배열 형식을 사용하여 룰을 지정할 필요가 있습니다.

<a name="rule-required"></a>
#### required

The field under validation must be present in the input data and not empty. A field is considered "empty" if one of the following conditions are true:

입력 값 중에 해당 필드가 존재해야 하며 비어 있어서는 안됩니다. 필드는 다음의 조건 중 하나를 충족하면 "빈(empty)" 것으로 간주됩니다: 

- The value is `null`.
- The value is an empty string.
- The value is an empty array or empty `Countable` object.
- The value is an uploaded file with no path.

- 값이 `null`인 경우.
- 값이 비어있는 문자열인 경우.
- 값이 비어있는 배열이거나, 비어있는 `Countable` 객체인경우 
- 값이 경로없이 업로드된 파일인 경우

<a name="rule-required-if"></a>
#### required_if:_anotherfield_,_value_,...

The field under validation must be present and not empty if the _anotherfield_ field is equal to any _value_.

만약 _anotherfield_의 값이 _value_중의 하나와 일치한다면, 해당 필드는 존재하고 비어있지 않아야 합니다.

<a name="rule-required-unless"></a>
#### required_unless:_anotherfield_,_value_,...

The field under validation must be present unless the _anotherfield_ field is equal to any _value_.
The field under validation must be present and not empty unless the _anotherfield_ field is equal to any _value_.

_anotherfield_가 어떤 _value_와도 값이 일치하지 않다면 해당 필드는 존재하고 비어있지 않아야 합니다. 

<a name="rule-required-with"></a>
#### required_with:_foo_,_bar_,...

The field under validation must be present and not empty _only if_ any of the other specified fields are present.

지정된 다른 필드중 하나라도 존재한다면, 해당 필드가 반드시 존재하고 비어있지 않아야 합니다.

<a name="rule-required-with-all"></a>
#### required_with_all:_foo_,_bar_,...

The field under validation must be present and not empty _only if_ all of the other specified fields are present.

지정된 다른 필드가 모두 존재한다면, 해당 필드가 반드시 존재하고 비어있지 않아야 합니다.

<a name="rule-required-without"></a>
#### required_without:_foo_,_bar_,...

The field under validation must be present and not empty _only when_ any of the other specified fields are not present.

지정된 다른 필드중 하나라도 존재하지 않으면, 해당 필드가 반드시 존재하고 비어있지 않아야 합니다.

<a name="rule-required-without-all"></a>
#### required_without_all:_foo_,_bar_,...

The field under validation must be present and not empty _only when_ all of the other specified fields are not present.

지정된 다른 필드들이 모두 존재하지 않으면, 해당 필드가 존재하고 비어있지 않아야 합니다.

<a name="rule-same"></a>
#### same:_field_

The given _field_ must match the field under validation.

필드의 값이 주어진 _field_의 값과 일치해야 합니다.

<a name="rule-size"></a>
#### size:_value_

The field under validation must have a size matching the given _value_. For string data, _value_ corresponds to the number of characters. For numeric data, _value_ corresponds to a given integer value. For an array, _size_ corresponds to the `count` of the array. For files, _size_ corresponds to the file size in kilobytes.

필드의 값이 주어진 _value_와 일치하는 크기를 가져야 합니다. 문자열 데이터에서는 문자의 개수가 _value_와 일치해야 합니다. 숫자형식의 데이터에서는 주어진 정수값이 _value_와 일치해야 합니다. 배열에서는 배열의 `count` 와 일치해야 합니다. 파일에서는 킬로바이트 형식의 파일 사이즈가 _size_와 일치해야 합니다.

<a name="rule-string"></a>
#### string

The field under validation must be a string.

필드의 값이 반드시 문자열이어야 합니다.

<a name="rule-timezone"></a>
#### timezone

The field under validation must be a valid timezone identifier according to the `timezone_identifiers_list` PHP function.

필드의 값이 `timezone_identifiers_list` PHP 함수에서 인식 가능한 유효한 timezone 식별자여야 합니다.

<a name="rule-unique"></a>
#### unique:_table_,_column_,_except_,_idColumn_

The field under validation must be unique on a given database table. If the `column` option is not specified, the field name will be used.

필드의 값이 주어진 데이터베이스 테이블에서 고유한 값이어야 합니다. 만약 `column`이 지정돼 있지 않다면 필드의 이름이 사용됩니다.

**Specifying A Custom Column Name:**
**특정 컬럼명 지정하기:**

    'email' => 'unique:users,email_address'

**Custom Database Connection**
**특정 데이터베이스 커넥션**

Occasionally, you may need to set a custom connection for database queries made by the Validator. As seen above, setting `unique:users` as a validation rule will use the default database connection to query the database. To override this, specify the connection followed by the table name using "dot" syntax:

때때로, 여러분은 Validator에 의해서 생성되는 데이터베이스 쿼리에 사용자가 지정한 커넥션을 필요로 할지도 모릅니다. 위에서의 검증 규칙 `unique:users` 에서는 데이터베이스를 쿼리하기 위해 기본 데이터 베이스 커넥션이 사용됩니다. 이를 재지정하려면 테이블 이름 후에 "." 표기법으로 커넥션을 지정하십시오:

    'email' => 'unique:connection.users,email_address'

**Forcing A Unique Rule To Ignore A Given ID:**
**주어진 ID에 대해서 유니크 규칙을 무시하도록 강제하기:**

Sometimes, you may wish to ignore a given ID during the unique check. For example, consider an "update profile" screen that includes the user's name, e-mail address, and location. Of course, you will want to verify that the e-mail address is unique. However, if the user only changes the name field and not the e-mail field, you do not want a validation error to be thrown because the user is already the owner of the e-mail address. You only want to throw a validation error if the user provides an e-mail address that is already used by a different user. To tell the unique rule to ignore the user's ID, you may pass the ID as the third parameter:

때때로 유니크 검사를 할 때 특정 ID를 무시하고자 할 수 있습니다. 예를 들어 사용자 이름, 이메일 주소 그리고 위치를 포함하는 "프로필 업데이트" 화면이 있습니다. 물론 이메일 주소가 고유하다는 것을 확인하고 싶을 것입니다. 하지만 사용자가 이름 필드만 바꾸고 이베일 필드를 바꾸지 않는다면 사용자가 이미 이메일 주소의 주인이기 때문에 유효 검사 오류가 던져지지 않아야 합니다. 유효 검사 오류는 사용자가 다른 사용자가 이미 사용하고 있는 이메일 주소를 제공할 때에만 던져져야 합니다. 유니크 규칙에 사용자 ID를 무시하라고 강제하기 위해서는 세번째 파라미터로 ID를 전달해야 합니다: 

    'email' => 'unique:users,email_address,'.$user->id

If your table uses a primary key column name other than `id`, you may specify it as the fourth parameter:

테이블이 `id`가 아닌 primary 키 컬럼 이름을 사용한다면 그 이름을 네번째 파라미터로 지정하면 됩니다: 

    'email' => 'unique:users,email_address,'.$user->id.',user_id'

**Adding Additional Where Clauses:**
**추가적인 Where 구문 추가하기:**

You may also specify more conditions that will be added as "where" clauses to the query:

더 추가되는 조건은 쿼리에 추가될 "where" 구문에 대해서 지정될 것입니다. 

    'email' => 'unique:users,email_address,NULL,id,account_id,1'

In the rule above, only rows with an `account_id` of `1` would be included in the unique check.

위의 룰에서는, `account_id`가 `1`인 데이터 만이 유니크 검사에 포함됩니다. 

<a name="rule-url"></a>
#### url

The field under validation must be a valid URL.

필드는 반드시 유효한 URL이어야 합니다. 

<a name="conditionally-adding-rules"></a>
## Conditionally Adding Rules
## 조건부로 룰 추가하기

In some situations, you may wish to run validation checks against a field **only** if that field is present in the input array. To quickly accomplish this, add the `sometimes` rule to your rule list:

어떤 상황에서는 필드가 입력 배열에 존재할 때에만 그 필드의 유효성 검사를 실행하고 싶을수도 있습니다. 이를 위해서는 룰의 목록에 `sometimes`를 추가하기만 하면 됩니다:

    $v = Validator::make($data, [
        'email' => 'sometimes|required|email',
    ]);

In the example above, the `email` field will only be validated if it is present in the `$data` array.

이 예제에서는 `$data` 배열에 `email` 필드가 존재할 경우에만 그 필드의 유효성 검사가 실행됩니다.

#### Complex Conditional Validation
#### 복잡한 조건부 유효성 검사

Sometimes you may wish to add validation rules based on more complex conditional logic. For example, you may wish to require a given field only if another field has a greater value than 100. Or, you may need two fields to have a given value only when another field is present. Adding these validation rules doesn't have to be a pain. First, create a `Validator` instance with your _static rules_ that never change:

때때로 여러분은 한가지 이상의 복잡한 로직에 대해서 유효성 규칙을 추가하고자 할 수도 있습니다. 예를 들어, 어떤 다른 필드가 100 이상의 값을 가질때에만 주어진 필드가 반드시 존재하길 바랄 수도 있습니다. 또는 다른 필드가 존재할 때에만 두개의 필드가 주어진 값을 가질 필요가 있을수도 있습니다. 이러한 유효성 검사 룰을 추가하는 것이 어렵지 않을 수 있습니다. 우선 _고정 룰_을 변경할 필요 없이 그대로 사용하여 `Validator` 인스턴스를 생성합니다:

    $v = Validator::make($data, [
        'email' => 'required|email',
        'games' => 'required|numeric',
    ]);

Let's assume our web application is for game collectors. If a game collector registers with our application and they own more than 100 games, we want them to explain why they own so many games. For example, perhaps they run a game re-sell shop, or maybe they just enjoy collecting. To conditionally add this requirement, we can use the `sometimes` method on the `Validator` instance.

여러분의 웹 어플리케이션이 게임 수집가들을 위한 사이트라고 가정해보겠습니다. 만약 100개 이상의 게임을 소유하고 있는 게임 수집가가 우리 사이트에 가입을 한다면, 우리는 그들이 왜 그렇게 많은 게임을 소유하고 있는지 설명을 듣고 싶을수 있습니다. 예를 들어, 아마 그들이 중고게임 판매점을 운영하거나, 단순히 수집을 취미로 할 수도 있습니다. 이런 요구사항을 조건부로 추가하기 위하여 `Validator` 인스턴스의 `sometimes` 메소드를 사용할 수 있습니다.

    $v->sometimes('reason', 'required|max:500', function($input) {
        return $input->games >= 100;
    });

The first argument passed to the `sometimes` method is the name of the field we are conditionally validating. The second argument is the rules we want to add. If the `Closure` passed as the third argument returns `true`, the rules will be added. This method makes it a breeze to build complex conditional validations. You may even add conditional validations for several fields at once:

`sometimes` 메소드에 전달되는 첫번째 인자는 필드의 이름입니다. 두번째 인자는 추가하려는 룰입니다. 만약 세번째 파라메터로 전달된 `Closure`가 `true`를 리턴한다면 그 룰은 유효성 검사에 추가될 것입니다. 이 메소드는 복잡한 조건부 유효성 검사의 구성을 손쉽게 만들어 주며, 한번에 여러개의 필드에 대한 조건부 유효성 검사를 추가할 수도 있습니다.

    $v->sometimes(['reason', 'cost'], 'required', function($input) {
        return $input->games >= 100;
    });

> **Note:** The `$input` parameter passed to your `Closure` will be an instance of `Illuminate\Support\Fluent` and may be used to access your input and files.

> **참고:** `Closure`로 전달된 `$input` 파라메터는 `Illuminate\Support\Fluent`의 인스턴스입니다. 그리고 입력된 데이터와 파일에 접근하기 위해 이 오브젝트가 사용할 수 있습니다.

<a name="custom-validation-rules"></a>
## Custom Validation Rules
## 사용자 정의 유효성 검사 룰

Laravel provides a variety of helpful validation rules; however, you may wish to specify some of your own. One method of registering custom validation rules is using the `extend` method on the `Validator` [facade](/docs/{{version}}/facades). Let's use this method within a [service provider](/docs/{{version}}/providers) to register a custom validation rule:

라라벨은 다양하고 유용한 유효성 검사 룰을 제공합니다; 하지만, 여러분은 여러분만의 유효성 검사 룰을 정의하길 바랄수도 있습니다. 커스텀 유효성 검사 룰을 등록하는 방법중 하나는 `Validator` [파사드](/docs/{{version}}/facades)에 `extend` 메소드를 사용하는 것입니다. 이 메소드를 [서비스 프로바이더](/docs/{{version}}/providers) 내에서 사용하여 커스텀 유효성 검사 룰을 등록합니다:

    <?php

    namespace App\Providers;

    use Validator;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            Validator::extend('foo', function($attribute, $value, $parameters, $validator) {
                return $value == 'foo';
            });
        }

        /**
         * Register the service provider.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

The custom validator Closure receives four arguments: the name of the `$attribute` being validated, the `$value` of the attribute, an array of `$parameters` passed to the rule, and the `Validator` instance.

커스텀 유효성 검사 클로저는 4개의 인자를 받습니다: 유효성 검사를 할 필드(`$attribute`)의 이름, 필드의 값(`$value`), 그리고 룰에 전달될 파라미터들(`$parameters`)의 배열, 그리고 `Validator` 인스턴스 입니다.  

You may also pass a class and method to the `extend` method instead of a Closure:

또한 클로저 대신 클래스명과 메소드명을 `extend` 메소드로 전달할 수도 있습니다.

    Validator::extend('foo', 'FooValidator@validate');

#### Defining The Error Message
#### 에러 메세지 정의하기

You will also need to define an error message for your custom rule. You can do so either using an inline custom message array or by adding an entry in the validation language file. This message should be placed in the first level of the array, not within the `custom` array, which is only for attribute-specific error messages:

커스텀 룰을 위한 에러 메세지 또한 정의해야 합니다. 인라인 커스텀 메세지 배열을 사용하거나 유효 검사 언어 파일에 엔트리를 추가하면 됩니다. 이 메세지는 `custom` 배열 안이 아닌 배열의 첫번째 레벨에 놓여야 합니다. `custom` 배열은 특정 속성에 따른 에러 메세지를 담당합니다:

    "foo" => "Your input was invalid!",

    "accepted" => "The :attribute must be accepted.",

    // The rest of the validation error messages...

When creating a custom validation rule, you may sometimes need to define custom place-holder replacements for error messages. You may do so by creating a custom Validator as described above then making a call to the `replacer` method on the `Validator` facade. You may do this within the `boot` method of a [service provider](/docs/{{version}}/providers):

커스텀 유효 검사 룰을 생성할 때 종종 에러 메세지를 위한 커스텀 플레이스 홀더 대체제를 정의해야 할 수도 있습니다. 이전의 설명에 따라 커스텀 Validator를 생성하고 `Validator` 파사드에 `replacer` 메소드를 호출하십시오. 이는 [서비스 프로바이더](/docs/{{version}}/providers)의 `boot` 메소드 안에서 할 수 있습니다:

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        Validator::extend(...);

        Validator::replacer('foo', function($message, $attribute, $rule, $parameters) {
            return str_replace(...);
        });
    }

#### Implicit Extensions
#### 묵시적 확장

By default, when an attribute being validated is not present or contains an empty value as defined by the [`required`](#rule-required) rule, normal validation rules, including custom extensions, are not run. For example, the [`unique`](#rule-unique) rule will not be run against a `null` value:

기본적으로 유효성 검사를 받는 속성이 존재하지 않거나 [`required`](#rule-required) 룰의 정의에 따라 빈 값을 가지고 있다면, 커스텀 확장을 포함한 정상적인 유효성 검사 룰은 실행되지 않을 것입니다. 예를 들어 `null` 값에는 [`unique`](#rule-unique) 룰이 실행되지 않을 것입니다: 

    $rules = ['name' => 'unique'];

    $input = ['name' => null];

    Validator::make($input, $rules)->passes(); // true

For a rule to run even when an attribute is empty, the rule must imply that the attribute is required. To create such an "implicit" extension, use the `Validator::extendImplicit()` method:

속성이 비었을 때도 룰이 실행되기 위해서는 룰에 속성이 필요하다는 것이 내포되어 있어야 합니다. 이런 "묵시적"인 확장을 만드려면 `Validator::extendImplicit()` 메소드를 사용하세요: 

    Validator::extendImplicit('foo', function($attribute, $value, $parameters, $validator) {
        return $value == 'foo';
    });

> **Note:** An "implicit" extension only _implies_ that the attribute is required. Whether it actually invalidates a missing or empty attribute is up to you.

> **주의:** "묵시적" 확장은 단지 속성이 필요하다는 것을 _암시(내포)_합니다. 없거나 빈 속성의 유효성을 실제로 부정하는지는 여러분이 결정합니다.