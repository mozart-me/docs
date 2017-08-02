# Валидация

- [Использование валидации](#basic-usage)
- [Работа с сообщениями об ошибках](#working-with-error-messages)
- [Сообщения об ошибках и представления](#error-messages-and-views)
- [Доступные правила валидации](#available-validation-rules)
- [Условные правила](#conditionally-adding-rules)
- [Собственные сообщения об ошибках](#custom-error-messages)
- [Собвственные правила проверки](#custom-validation-rules)

<a name="basic-usage"></a>
## Использование валидации

Октябрь поставляется с простой, удобной системой валидации (проверки входных данных на соответствие правилам) и получения сообщений об ошибках - классом `Validator`.

#### Простейший пример валидации

    $validator = Validator::make(
        ['name' => 'Joe'],
        ['name' => 'required|min:5']
    );

Первый параметр, передаваемый методу `make` - данные для проверки. Второй параметр - правила, которые к ним должны быть применены.

#### Использование массивов для указания правил

Несколько правил могут быть разделены либо прямой чертой (|), либо быть отдельными элементами массива.

    $validator = Validator::make(
        ['name' => 'Joe'],
        ['name' => ['required', 'min:5']]
    );

#### Проверка нескольких полей

    $validator = Validator::make(
        [
            'name' => 'Joe',
            'password' => 'lamepassword',
            'email' => 'email@example.com'
        ],
        [
            'name' => 'required',
            'password' => 'required|min:8',
            'email' => 'required|email|unique:users'
        ]
    );

Как только был создан экземпляр `Validator`, метод `fails` (или `passes`) может быть использован для проведения проверки.

    if ($validator->fails()) {
        // Переданные данные не прошли проверку
    }

Если `Validator` нашёл ошибки, то Вы можете получить его сообщения таким образом:

    $messages = $validator->messages();

Вы также можете получить массив правил, данные которые не прошли проверку, без самих сообщений:

    $failed = $validator->failed();

#### Проверка файлов

Класс `Validator` содержит несколько изначальных правил для проверки файлов, такие как `size`, `mimes` и другие. Для выполнения проверки над файлами просто передайте эти файлы вместе с другими данными.

<a name="working-with-error-messages"></a>
## Работа с сообщениями об ошибках

После вызова метода `messages` объекта `Validator` Вы получите экземпляр `Illuminate\Support\MessageBag`, который имеет набор полезных методов для доступа к сообщеням об ошибках.

#### Получение первого сообщения для поля

    echo $messages->first('email');

#### Получение всех сообщений для одного поля

    foreach ($messages->get('email') as $message) {
        //
    }

#### Получение всех сообщений для всех полей

    foreach ($messages->all() as $message) {
        //
    }

#### Проверка на наличие сообщения для поля

    if ($messages->has('email')) {
        //
    }

#### Получение ошибки в заданном формате

    echo $messages->first('email', '<p>:message</p>');

> **Примечание:** По умолчанию сообщения отформатированы согласно синтаксису Bootstrap.

#### Получение все сообщений в заданном формате

    foreach ($messages->all('<li>:message</li>') as $message) {
        //
    }

<a name="error-messages-and-views"></a>
## Сообщения об ошибках и представления

Как только вы провели проверку, вам понадобится простой способ, чтобы передать ошибки в шаблон. Октбярь позволяет удобно сделать это. Например, у нас есть такие роуты:

    public function onRegister()
    {
        $rules = [];

        $validator = Validator::make(Input::all(), $rules);

        if ($validator->fails()) {
            return Redirect::to('register')->withErrors($validator);
        }
    }

Заметьте, что когда проверки не пройдены, мы передаём объект `Validator` объекту переадресации Redirect при помощи метода `withErrors`. Этот метод сохранит сообщения об ошибках в одноразовых flash-переменных сессии, таким образом делая их доступными для следующего запроса.

Октябрь постоянно проверяет данные сессии на наличие ошибок и, если они существуют, то  автоматически привязывает их к представлению. **Таким образом, важно помнить, что переменная `errors` будет доступна для всех ваших страниц всегда и при любом запросе**. Это позволяет вам считать, что переменная `errors` всегда определена и может безопасно использоваться. Переменная `errors` - экземпляр класса `MessageBag`.

Таким образом, после переадресации вы можете прибегнуть к автоматически установленной в шаблоне переменной `errors`:

    {{ errors.first('email') }}

### Именованные MessageBag

Если у вас есть несколько форм на странице, то вы можете выбрать имя объекта `MessageBag`, в котором будут возвращаться тексты ошибок, чтобы Вы могли их корректно отобразить для нужной формы. Просто передайте имя в качестве второго аргумента в `withErrors`:

    return Redirect::to('register')->withErrors($validator, 'login');

Получить текст ошибки из `MessageBag` с именем `login`:

    {{ errors.login.first('email') }}

<a name="available-validation-rules"></a>
## Доступные правила валидации

Ниже приведен список всех доступных правил и их функции:

<style>
    .collection-method-list {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        column-gap: 2em; -moz-column-gap: 2em; -webkit-column-gap: 2em;
    }

    .collection-method-list a {
        display: block;
    }
</style>

<div class="content-list collection-method-list" markdown="1">
- [Accepted](#rule-accepted)
- [Active URL](#rule-active-url)
- [After (Date)](#rule-after)
- [Alpha](#rule-alpha)
- [Alpha Dash](#rule-alpha-dash)
- [Alpha Numeric](#rule-alpha-num)
- [Array](#rule-array)
- [Before (Date)](#rule-before)
- [Between](#rule-between)
- [Boolean](#rule-boolean)
- [Confirmed](#rule-confirmed)
- [Date](#rule-date)
- [Date Format](#rule-date-format)
- [Different](#rule-different)
- [Digits](#rule-digits)
- [Digits Between](#rule-digits-between)
- [E-Mail](#rule-email)
- [Exists (Database)](#rule-exists)
- [Image (File)](#rule-image)
- [In](#rule-in)
- [Integer](#rule-integer)
- [IP Address](#rule-ip)
- [Max](#rule-max)
- [MIME Types](#rule-mimes)
- [Min](#rule-min)
- [Not In](#rule-not-in)
- [Numeric](#rule-numeric)
- [Regular Expression](#rule-regex)
- [Required](#rule-required)
- [Required If](#rule-required-if)
- [Required With](#rule-required-with)
- [Required With All](#rule-required-with-all)
- [Required Without](#rule-required-without)
- [Required Without All](#rule-required-without-all)
- [Same](#rule-same)
- [Size](#rule-size)
- [String](#rule-string)
- [Timezone](#rule-timezone)
- [Unique (Database)](#rule-unique)
- [URL](#rule-url)
</div>

<a name="rule-accepted"></a>
#### accepted

Поле должно быть в значении _yes_, _on_ или _1_. Это полезно для проверки принятия правил и лицензий.

<a name="rule-active-url"></a>
#### active_url

Поле должно быть корректным URL, доступным через PHP функцию `checkdnsrr`.

<a name="rule-after"></a>
#### after:_date_

Поле должно быть датой, более поздней, чем date. Строки приводятся к датам PHP функцией `strtotime`.

<a name="rule-alpha"></a>
#### alpha

Поле можно содержать только только латинские символы.

<a name="rule-alpha-dash"></a>
#### alpha_dash

Поле можно содержать только латинские символы, цифры, знаки подчёркивания (\_) и дефисы (-).

alpha_num
<a name="rule-alpha-num"></a>
#### alpha_num

Поле можно содержать только латинские символы и цифры.

<a name="rule-array"></a>
#### array

Поле должно быть массивом.

<a name="rule-before"></a>
#### before:_date_

Поле должно быть датой, более ранней, чем date. Строки приводятся к датам PHP функцией `strtotime`.

<a name="rule-between"></a>
#### between:_min_,_max_

Поле должно быть числом в диапазоне от min до max. Строки, числа и файлы трактуются аналогично правилу `size`.

<a name="rule-boolean"></a>
#### boolean

Поле должно быть логическим (булевым). Разрешенные значения: `true`, `false`, `1`, `0`, `"1"` и `"0"`.

<a name="rule-confirmed"></a>
#### confirmed

Значение поля должно соответствовать значению поля с этим именем, плюс `foo_confirmation`. Например, если проверяется поле `password`, то на вход должно быть передано совпадающее по значению поле `password_confirmation`.

<a name="rule-date"></a>
#### date

Поле должно быть правильной датой в соответствии с PHP функцией `strtotime`.

<a name="rule-date-format"></a>
#### date_format:_format_

Поле должно подходить под формату даты format в соответствии с PHP функцией `date_parse_from_format`.

<a name="rule-different"></a>
#### different:_field_

Значение проверяемого поля должно отличаться от значения поля _field_.

<a name="rule-digits"></a>
#### digits:_value_

Значение проверяемого поля должно быть числом и иметь точную длину _value_.

<a name="rule-digits-between"></a>
#### digits_between:_min_,_max_

The field under validation must have a length between the given _min_ and _max_.

<a name="rule-email"></a>
#### email

The field under validation must be formatted as an e-mail address.

<a name="rule-exists"></a>
#### exists:_table_,_column_

The field under validation must exist on a given database table.

#### Basic usage of exists rule

    'state' => 'exists:states'

#### Specifying a custom column name

    'state' => 'exists:states,abbreviation'

You may also specify more conditions that will be added as "where" clauses to the query:

    'email' => 'exists:staff,email,account_id,1'

Passing `NULL` as a "where" clause value will add a check for a `NULL` database value:

    'email' => 'exists:staff,email,deleted_at,NULL'

<a name="rule-image"></a>
#### image

The file under validation must be an image (jpeg, png, bmp, or gif)

<a name="rule-in"></a>
#### in:_foo_,_bar_,...

The field under validation must be included in the given list of values.

<a name="rule-integer"></a>
#### integer

The field under validation must have an integer value.

<a name="rule-ip"></a>
#### ip

The field under validation must be formatted as an IP address.

<a name="rule-max"></a>
#### max:_value_

The field under validation must be less than or equal to a maximum _value_. Strings, numerics, and files are evaluated in the same fashion as the [`size`](#rule-size) rule.

<a name="rule-mimes"></a>
#### mimes:_foo_,_bar_,...

The file under validation must have a MIME type corresponding to one of the listed extensions.

#### Basic usage of MIME rule

    'photo' => 'mimes:jpeg,bmp,png'

<a name="rule-min"></a>
#### min:_value_

The field under validation must have a minimum _value_. Strings, numerics, and files are evaluated in the same fashion as the [`size`](#rule-size) rule.

<a name="rule-not-in"></a>
#### not_in:_foo_,_bar_,...

The field under validation must not be included in the given list of values.

<a name="rule-numeric"></a>
#### numeric

The field under validation must have a numeric value.

<a name="rule-regex"></a>
#### regex:_pattern_

The field under validation must match the given regular expression.

**Note:** When using the `regex` pattern, it may be necessary to specify rules in an array instead of using pipe delimiters, especially if the regular expression contains a pipe character.

<a name="rule-required"></a>
#### required

The field under validation must be present in the input data.

<a name="rule-required-if"></a>
#### required_if:_field_,_value_,...

The field under validation must be present if the _field_ field is equal to any _value_.

<a name="rule-required-with"></a>
#### required_with:_foo_,_bar_,...

The field under validation must be present _only if_ any of the other specified fields are present.

<a name="rule-required-with-all"></a>
#### required_with_all:_foo_,_bar_,...

The field under validation must be present _only if_ all of the other specified fields are present.

<a name="rule-required-without"></a>
#### required_without:_foo_,_bar_,...

The field under validation must be present _only when_ any of the other specified fields are not present.

<a name="rule-required-without-all"></a>
#### required_without_all:_foo_,_bar_,...

The field under validation must be present _only when_ the all of the other specified fields are not present.

<a name="rule-same"></a>
#### same:_field_

The specified _field_ value must match the field's value under validation.

<a name="rule-size"></a>
#### size:_value_

The field under validation must have a size matching the given _value_. For string data, _value_ corresponds to the number of characters. For numeric data, _value_ corresponds to a given integer value. For files, _size_ corresponds to the file size in kilobytes.

<a name="rule-string"></a>
#### string:_value_

The field under validation must be a string type.

<a name="rule-timezone"></a>
#### timezone

The field under validation must be a valid timezone identifier according to the `timezone_identifiers_list` PHP function.

<a name="rule-unique"></a>
#### unique:_table_,_column_,_except_,_idColumn_

The field under validation must be unique on a given database table. If the `column` option is not specified, the field name will be used.

#### Basic usage of unique rule

    'email' => 'unique:users'

#### Specifying a custom column name

    'email' => 'unique:users,email_address'

#### Forcing a unique rule to ignore a given ID

    'email' => 'unique:users,email_address,10'

#### Adding additional where clauses

You may also specify more conditions that will be added as "where" clauses to the query:

    'email' => 'unique:users,email_address,NULL,id,account_id,1'

In the rule above, only rows with an `account_id` of `1` would be included in the unique check.

<a name="rule-url"></a>
#### url

The field under validation must be formatted as an URL.

> **Note:** This function uses PHP's `filter_var` method.

<a name="conditionally-adding-rules"></a>
## Conditionally adding rules

In some situations, you may wish to run validation checks against a field **only** if that field is present in the input array. To quickly accomplish this, add the `sometimes` rule to your rule list:

    $v = Validator::make($data, [
        'email' => 'sometimes|required|email',
    ]);

In the example above, the `email` field will only be validated if it is present in the `$data` array.

#### Complex conditional validation

Sometimes you may wish to require a given field only if another field has a greater value than 100. Or you may need two fields to have a given value only when another field is present. Adding these validation rules doesn't have to be a pain. First, create a `Validator` instance with your _static rules_ that never change:

    $v = Validator::make($data, [
        'email' => 'required|email',
        'games' => 'required|numeric',
    ]);

Let's assume our web application is for game collectors. If a game collector registers with our application and they own more than 100 games, we want them to explain why they own so many games. For example, perhaps they run a game re-sell shop, or maybe they just enjoy collecting. To conditionally add this requirement, we can use the `sometimes` method on the `Validator` instance.

    $v->sometimes('reason', 'required|max:500', function($input) {
        return $input->games >= 100;
    });

The first argument passed to the `sometimes` method is the name of the field we are conditionally validating. The second argument is the rules we want to add. If the `Closure` passed as the third argument returns `true`, the rules will be added. This method makes it a breeze to build complex conditional validations. You may even add conditional validations for several fields at once:

    $v->sometimes(['reason', 'cost'], 'required', function($input) {
        return $input->games >= 100;
    });

> **Note:** The `$input` parameter passed to your `Closure` will be an instance of `Illuminate\Support\Fluent` and may be used as an object to access your input and files.

<a name="custom-error-messages"></a>
## Custom error messages

If needed, you may use custom error messages for validation instead of the defaults. There are several ways to specify custom messages.

#### Passing custom messages into validator

    $messages = [
        'required' => 'The :attribute field is required.',
    ];

    $validator = Validator::make($input, $rules, $messages);

> *Note:* The `:attribute` place-holder will be replaced by the actual name of the field under validation. You may also utilize other place-holders in validation messages.

#### Other validation placeholders

    $messages = [
        'same'    => 'The :attribute and :other must match.',
        'size'    => 'The :attribute must be exactly :size.',
        'between' => 'The :attribute must be between :min - :max.',
        'in'      => 'The :attribute must be one of the following types: :values',
    ];

#### Specifying a custom message for a given attribute

Sometimes you may wish to specify a custom error messages only for a specific field:

    $messages = [
        'email.required' => 'We need to know your e-mail address!',
    ];

<a name="localization"></a>
#### Specifying custom messages in language files

In some cases, you may wish to specify your custom messages in a language file instead of passing them directly to the `Validator`. To do so, add your messages to an array in the `lang/xx/validation.php` language file for your plugin.

    return  [
        'required' => 'We need to know your e-mail address!',
        'email.required' => 'We need to know your e-mail address!',
    ];

Then in your call to `Validator::make` use the `Lang:get` to use your custom files.

    Validator::make($formValues, $validations, Lang::get('acme.blog::validation'));

<a name="custom-validation-rules"></a>
## Custom validation rules

#### Registering a custom validation rule

There are a variety of helpful validation rules; however, you may wish to specify some of your own. One method of registering custom validation rules is using the `Validator::extend` method:

    Validator::extend('foo', function($attribute, $value, $parameters) {
        return $value == 'foo';
    });

The custom validator Closure receives three arguments: the name of the `$attribute` being validated, the `$value` of the attribute, and an array of `$parameters` passed to the rule.

You may also pass a class and method to the `extend` method instead of a Closure:

    Validator::extend('foo', 'FooValidator@validate');

Note that you will also need to define an error message for your custom rules. You can do so either using an inline custom message array or by adding an entry in the validation language file.

#### Extending the validator class

Instead of using Closure callbacks to extend the Validator, you may also extend the Validator class itself. To do so, write a Validator class that extends `Illuminate\Validation\Validator`. You may add validation methods to the class by prefixing them with `validate`:

    <?php

    class CustomValidator extends Illuminate\Validation\Validator
    {

        public function validateFoo($attribute, $value, $parameters)
        {
            return $value == 'foo';
        }

    }

#### Registering a custom validator resolver

Next, you need to register your custom Validator extension:

    Validator::resolver(function($translator, $data, $rules, $messages, $customAttributes) {
        return new CustomValidator($translator, $data, $rules, $messages, $customAttributes);
    });

When creating a custom validation rule, you may sometimes need to define custom placeholder replacements for error messages. You may do so by creating a custom Validator as described above, and adding a `replaceXXX` function to the validator.

    protected function replaceFoo($message, $attribute, $rule, $parameters)
    {
        return str_replace(':foo', $parameters[0], $message);
    }

If you would like to add a custom message "replacer" without extending the `Validator` class, you may use the `Validator::replacer` method:

    Validator::replacer('rule', function($message, $attribute, $rule, $parameters) {
        //
    });