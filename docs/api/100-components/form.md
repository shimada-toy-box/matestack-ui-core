# Matestack Core Component: Form


The `form` core component is a Vue.js driven component. It enables you to implement dynamic forms without writing JavaScript. It relies on child components to collect and submit user input: `form_input`, `form_textarea`, `form_radio`, `form_select`, `form_checkbox` and `form_submit`. They are described within this documentation page.

## Parameters

The core form component accepts the following parameters. Pass them in as a hash like so:

```ruby
class ExamplePage < Matestack::Ui::Page

  def response
    form some_form_config do
      #...
    end
  end

  def some_form_config
    {
      for: :my_object,
      method: :post,
      path: :success_form_test_path,
      #...
    }
  end

end
```

### For

The `form` component wraps the input in an object. The name of this object can be set in multiple ways:

#### set as a symbol

```ruby
for: :my_object
```

```ruby
form some_form_config do
  form_input key: :some_input_key, type: :text
end
```

When submitting this form, the `form` component will perform a request with a payload like this:

```ruby
my_object: {
  some_input_key: "foo"
}
```

#### set by Active Record class name

```ruby
@my_model = MyActiveRecordModel.new
#...
for: @my_model
```

```ruby
form some_form_config do
  form_input key: :some_model_attribute, type: :text
end
```

When submitting this form, the `form` component will perform a request with a payload like this:

```ruby
my_active_record_model: {
  some_model_attribute: "foo"
}
```

Please be aware that if you use an Active Record model, the keys of the input components should match a model attribute/method. The form automatically tries to prefill the form inputs through calling the keys as methods on the model.

### Method

This specifies which kind of HTTP method should get triggered. It accepts a symbol like so:

```ruby
method: :post
```

### Path

This parameter accepts a classic Rails path, usually in the form of a symbol like so:

```ruby
path: :action_test_path
```

or

```ruby
path: action_test_path(id: 42)
```
if you want to use rails url helper methods.

### Params

Using the standard Rails params, we can pass params to our route. Only applies when `path` is given as a symbol.

```ruby
params: { id: 42 }
```

### Emit

This event gets emitted right after form submit. In contrast to the `success` or `failure` events, it will be emitted regardless of the server response.

```ruby
emit: "form_submitted"
```

### Delay

You can use this attribute if you want to delay the actual form submit request. It will not delay the event specified with the `emit` attribute.

```ruby
delay: 1000 # means 1000 ms
```

### Multipart

If you want to perform file uploads within this form, you have to set `multipart` to true. It will send the form data with `"Content-Type": "multipart/form-data"`

```ruby
multipart: true # default is false which results in form submission via "Content-Type": "application/json"
```

### Success

The success part of the `form` component gets triggered once the controller action we wanted to call returns a success code, usually the `2xx` HTTP status code.


#### Emit event

To trigger further behavior, we can configure the success part of a `form` to emit a message like so:

```ruby
success: {
  emit: 'my_form_success'
}
```

#### Perform transition

We can also perform a transition that only gets triggered on success and also accepts further params:

```ruby
success: {
  emit: 'my_form_success',
  transition: {
    path: :form_test_page2_path,
    params: { id: 42 }
  }
}
```

When the server redirects to a url, for example after creating a new record, the transition needs to be configured to follow this redirect of the server response.

```ruby
success: {
  emit: 'my_form_success',
  transition: {
    follow_response: true
  }
}
```

A controller action that would create a record and then respond with the url the page should transition to, could look like this:

```ruby
class TestModelsController < ApplicationController
  include Matestack::Ui::Core::ApplicationHelper

  def create
    @test_model = TestModel.create(test_model_params)

    render json: {
      transition_to: test_model_path(@test_model)
    }, status: :ok
  end
end
```

Same applies for the `failure` configuration.

#### Perform redirect

We can also perform a redirect (full page load) that only gets triggered on success and also accepts further params:

Please be aware, that emiting a event doen't have an effect when performing a redirect instead of a transition, as the whole page (including the surrounding app) gets reloaded!

```ruby
success: {
  emit: 'my_form_success', # doesn't have an effect when using redirect
  redirect: {
    path: :action_test_page2_path,
    params: { id: 42 }
  }
}
```

When the server redirects to a url, for example after creating a new record, the redirect needs to be configured to follow this redirect of the server response.

```ruby
success: {
  emit: 'my_form_success', # doesn't have an effect when using redirect
  redirect: {
    follow_response: true
  }
}
```

A controller action that would create a record and then respond with the url the page should redirect to, could look like this:

```ruby
class TestModelsController < ApplicationController
  include Matestack::Ui::Core::ApplicationHelper

  def create
    @test_model = TestModel.create(test_model_params)

    render json: {
      redirect_to: test_model_path(@test_model)
    }, status: :ok
  end
end
```

Same applies for the `failure` configuration.

#### Reset form

If submitted successfully, the `form` component resets its state by default when using the "post" method. When using the "put" method, the state is not resetted by default. You may control this behavior explictly by using the `reset` option:

```ruby
method: :post
success: {
  emit: 'my_form_success',
  reset: false #default true when using the :post method when submitted successfully
}
```

```ruby
method: :put
success: {
  emit: 'my_form_success',
  reset: true #default false when using the :put method when submitted successfully
}
```


### Failure

As counterpart to the success part of the `form` component, there is also the possibility to define the failure behavior. This is what gets triggered after the response to our `form` submit returns a failure code, usually in the range of `400` or `500` HTTP status codes.

#### Emit event

To trigger further behavior, we can configure the failure part of an action to emit a message like so:

```ruby
failure: {
  emit: 'my_form_failure'
}
```


#### Perform transition

We can also perform a transition that only gets triggered on failure:

```ruby
failure: {
  emit: 'my_form_failure',
  transition: {
    path: :root_path
  }
}
```

#### Reset form

The `form` component does not reset its state by default when not submitted successfully. You may control this behavior explictly by using the `reset` option:

```ruby
method: :post
failure: {
  emit: 'my_form_failure',
  reset: true #default false when using the :post method and not successful
}
```

```ruby
method: :put
failure: {
  emit: 'my_form_failure',
  reset: true #default false when using the :put method and not successful
}
```

#### ID - optional

This parameter accepts a string of ids that the form component should have:

```ruby
id: 'my-form-id'
```

which renders as an HTML `id` attribute, like so:

```html
<form id="my-form-id" class="matestack-form">...</form>
```

#### Class - optional

This parameter accepts a string of classes that the form component should have:

```ruby
class: 'my-form-class'
```

which renders as an HTML `class` attribute, like so:

```html
<form class="matestack-form my-form-class">...</form>
```

## Child components

### form_input

```ruby
form my_form_config do
  form_input key: :foo, type: :text, #...
end
```

#### Parameters

##### key - required

Defines the key which should be used when posting the form data to the server.

##### type - required

Pass in as symbol. Defines the type of the `input`. All HTML input types are supported.

##### placeholder - optional

Defines the placeholder.

##### label - optional

Defines the label which will be rendered right before the input tag. You can also use the `label` component in order to create more complex label structures.

##### required - optional

If set to true, HTML validations will be triggered on form submit.

##### init - optional

If given, this value will be the initial value of the input. If used in an edit form of a given ActiveRecord model, the init value will be automatically derived from the model itself.

### form_textarea

```ruby
form my_form_config do
  form_textarea key: :foo, #...
end
```

#### Parameters

##### key - required

Defines the key which should be used when posting the form data to the server.

##### placeholder - optional

Defines the placeholder.

##### label - optional

Defines the label which will be rendered right before the textarea tag. You can also use the `label` component in order to create more complex label structures.

##### required - optional

If set to true, HTML validations will be triggered on form submit.

##### init - optional

If given, this value will be the initial value of the input. If used in an edit form of a given ActiveRecord model, the init value will be automatically derived from the model itself.

### form_select

```ruby
form my_form_config do
  form_select key: :status, options: { 'active': 1, 'deactive': 0 }, #...
end
```

#### Parameters

##### key - required

Defines the key which should be used when posting the form data to the server.

##### options - required

Can either be an Array or Hash:

**Array usage**

```ruby
form my_form_config do
  form_select key: :status, options: [0, 1]
end
```

will render:

```html
<select>
  <option value="0">0</option>
  <option value="1">1</option>
</select>
```

**Hash usage**

```ruby
form my_form_config do
  form_select key: :status, options: { 'active': 1, 'deactive': 0 }
end
```

will render:

```html
<select>
  <option value="0">deactive</option>
  <option value="1">active</option>
</select>
```

The hash values will be used as values for the options, the keys as displayed values.

**ActiveRecord Enum Mapping**

If you want to use ActiveRecord enums as options for your select input, you can use the enum class method:

```ruby
class Conversation < ActiveRecord::Base
  enum status: { active: 0, archived: 1 }
end
```

```ruby
form my_form_config do
  form_select key: :status, options: Conversation.statuses
end
```

##### disabled_values - optional

Defines which options (by value) should be disabled. Pass in as Array.

```ruby
form my_form_config do
  form_select key: :status, options: [1,2,3], disabled_values: [1, 2]
end
```

or

```ruby
form my_form_config do
  form_select key: :status, options: { 'active': 1, 'deactive': 0 }, disabled_values: [1]
end
```

##### multiple - optional

If set to true, a native HTML multiple select will be rendered. Selected values will be posted as an Array to the server.

##### init - optional

Defines the init value of the select input. If mapped to an ActiveRecord model, the init value will be derived automatically from the model instance.

**when multiple is set to true**

Pass in an Array of init values:

```ruby
form my_form_config do
  form_select key: :status, [1,2,3], multiple: true, init: [1,2]
end
```

**when multiple is set to false/not specified**

Pass in an Integer:

```ruby
form my_form_config do
  form_select key: :status, [1,2,3], init: 1
end
```

##### placeholder - optional

Defines the placeholder which will be rendered as first, disabled option.

##### label - optional

Defines the label which will be rendered right before the textarea tag. You can also use the `label` component in order to create more complex label structures.

### form_radio

```ruby
form my_form_config do
  form_radio key: :status, options: { 'active': 1, 'deactive': 0 }, #...
end
```

#### Parameters

##### key - required

Defines the key which should be used when posting the form data to the server.

##### options - required

Can either be an Array or Hash:

**Array usage**

```ruby
form my_form_config do
  form_radio key: :status, options: [0, 1]
end
```

will render:

```html
<input id="status_0" name="status_0" type="radio" value="0">
<label for="status_0">
0
</label>
<input id="status_1" name="status_1" type="radio" value="1">
<label for="status_1">
1
</label>
```

**Hash usage**

```ruby
form my_form_config do
  form_select key: :status, options: { 'active': 1, 'deactive': 0 }
end
```

will render:

```html
<input id="status_1" name="status_active" type="radio" value="1">
<label for="status_1">
  active
</label>
<input id="status_0" name="status_deactive" type="radio" value="0">
<label for="status_0">
  deactive
</label>
```

The hash values will be used as values for the options, the keys as displayed values.

**ActiveRecord Enum Mapping**

If you want to use ActiveRecord enums as options for your radio input, you can use the enum class method:

```ruby
class Conversation < ActiveRecord::Base
  enum status: { active: 0, archived: 1 }
end
```

```ruby
form my_form_config do
  form_radio key: :status, options: Conversation.statuses
end
```

##### disabled_values - optional

NOT IMPLEMENTED YET

##### init - optional

Defines the init value of the radio input. If mapped to an ActiveRecord model, the init value will be derived automatically from the model instance.

Pass in an Integer:

```ruby
form my_form_config do
  form_radio key: :status, [1,2,3], init: 1
end
```

##### placeholder - optional

Defines the placeholder which will be rendered as first, disabled option.

##### label - optional

NOT IMPLEMENTED YET

You can also use the `label` component in order to create a label for this input.

### form_checkbox

```ruby
form my_form_config do
  form_checkbox key: :status, #...
end
```

#### Parameters

##### key - required

Defines the key which should be used when posting the form data to the server.

##### options - optional

Can either be nil, an Array or Hash:

**When not given**

```ruby
form my_form_config do
  form_checkbox key: :status, label: "Active"
end
```

will render a single checkbox which can switch between `true` and `false` as value for the given key. Will be `nil` initially. The boolean value (or nil) will be sent to the server when submitting the form.

**Array usage**

```ruby
form my_form_config do
  form_checkbox key: :status, options: [0, 1]
end
```

will render a collection of checkboxes and their corresponding labels.

Multiple checkboxes can be selected. Data will be sent as an Array of selected values to the server when submitting the form.

**Hash usage**

```ruby
form my_form_config do
  form_checkbox key: :status, options: { 'active': 1, 'deactive': 0 }
end
```

will render a collection of checkboxes and their corresponding labels.

The hash values will be used as values for the checkboxes, the keys as displayed label values.

Multiple checkboxes can be selected. Data will be sent as an Array of selected values to the server when submitting the form.


**ActiveRecord Enum Mapping**

If you want to use ActiveRecord enums as options for your radio input, you can use the enum class method:

```ruby
class Conversation < ActiveRecord::Base
  enum status: { active: 0, archived: 1 }
end
```

```ruby
form my_form_config do
  form_checkbox key: :status, options: Conversation.statuses
end
```

Multiple checkboxes can be selected. Data will be sent as an Array of selected values to the server when submitting the form.


##### disabled_values - optional

NOT IMPLEMENTED YET

##### init - optional

NOT IMPLEMENTED YET

##### label - optional

An applied label is only visible, when using a single checkbox without options.

You can also use the `label` component in order to create a label for this input.

### form_submit

```ruby
form my_form_config do
  #...
  form_submit do
    button text: "submit"
  end
end
```

`form_submit` does not take any parameters but yields a given block. This block will be wrapped by a span tag. A click on that span tag and all elements within will submit the form.


### Special inputs

#### File Upload

In order to perform a single file upload, add this `form_input` component

```ruby
form_input key: :some_file, type: :file
```

In order to perform multiple file uploads, add this `form_input` component

```ruby
form_input key: :some_files, type: :file, multiple: true
```

Don't forget to add the `multiple: true` attribute to your `form_config`!

In order to accept multiple files, you should permit params on your controller like:

`some_controller.rb`

```ruby
#...
params.require(:my_form_wrapper_key).permit(
  :some_file,
  some_files: []
)
#...
```

Please be aware that the `form_input` components with a `type: :file` can not be initialized with a given file.

#### Range Input

Whilst working similiar to the 'text' input, the range input accepts a few more parameters. It accepts also 'min', 'max', 'step', 'list' as optional parameters.

**with max, min, step set**

```ruby
form_input id: 'range-input', type: :range, min: 0, max: 100, step: 1
```

**with corresponding datalist**

To use a datalist for the range input specify the 'list' parameter with the id of the provided datalist

```ruby
form_input id: 'range-input', type: :range, list: 'datalist-id'
datalist id: 'datalist-id' do
  option value: 10
  option value: 20
end
```

## Error rendering

If the server is responding with a well formatted error response and status after submitting the form, matestack will automatically render server error messages right next to the corresponding input (matching error and input key). Additionally the input itself will get a 'error' css class; the parent form will get a 'has-errors' css class.

The described approach is suitable for all form_* input components.

By default it would look like this:

```ruby
form_input key: :title, type: :text
```

```html
<form class="matestack-form has-errors">
  <input type="text" class="error">
  <span class="errors">
    <!-- for each error string within the error array for the key 'title' -->
    <span class="error">
      can't be blank
    </span>
  </span>
</form>
```

when a `4xx` JSON server response like that was given (ActiveRecord errors format):

```json
{
  "errors":
    {
      "title": ["can't be blank"]
    }
}
```

You can modify the error rendering like this:

**On input level**

```ruby
form_input key: :foo, type: :text,  errors: {
  wrapper: { tag: :div, class: 'my-errors'}, tag: :div, class: 'my-error'
}
form_input key: :bar, type: :text,  errors: false
```

```html
<form class="matestack-form has-errors">
  <input type="text" class="error">
  <div class="my-errors">
    <!-- for each error string within the error array for the key 'title' -->
    <div class="my-error">
      can't be blank
    </div>
  </div>
  <input type="text" class="error">
  <!-- without any error rendering because its set to false -->
</form>
```

**On form level**

Configuring errors on a per form basis. Per form field configs take precedence over the form config.

```ruby
def response
  form form_config do
    form_input key: :foo, type: :text
    form_input key: :bar, type: :text, errors: false
  end
end

def form_config
  {
    for: :my_model,
    #[...]
    errors: {
      wrapper: { tag: :div, class: 'my-errors' },
      input: { class: 'my-error' },
      tag: :div,
      class: 'my-field-error'
    }
  }
```

Outputs errors as:

```html
<input type="text" class="my-field-error" />
<div class="my-errors">
  <div class="my-error">
    can't be blank
  </div>
</div>
<input type="text" class="my-field-error" />
<!-- without any errors, because its config takes precedence over the form config -->
```

## Loading state

The form will get a 'loading' css class while submitting the form and waiting for a server response:

```html
<form class="matestack-form loading">

</form>
```

Additionally the `span` tag rendered by the `form_submit` component around your submit components will also get a 'loading' css class.

If you simply want to disable your submit button, you can use a simple Vue.js binding:

```ruby
form_submit do
  button text: "Submit me!", attributes: { "v-bind:disabled": "loading" }
end

```

If you want to adjust the submit element more flexible while the form is being submitted, you could use the event mechanism of the form in combination with the `toggle` component:

```ruby
form_submit do
  toggle hide_on: "form_loading", show_on: "form_succeeded, form_failed", init_show: true do
    button text: "Submit me!"
  end
  toggle show_on: "form_loading", hide_on: "form_succeeded, form_failed" do
    button text: "submitting...", disabled: true
  end
end
```

in combination with a form config like this:

```ruby
def my_form_config
  {
    #...
    emit: "form_loading",
    success: {
      emit: "form_succeeded"
    },
    failure: {
      emit: "form_failed"
    }
  }
end

```

## Form and other VueJs components

The child components `form_*` have to be placed within the scope of the parent `form` component, without any other VueJs component like `toggle`, `async` creating a new scope between the child component and the parent form component**


```ruby
# that's working:
form some_form_config do
  form_input key: :some_input_key, type: :text
  toggle show_on: "some-event" do
    plain "hello!"
  end
end

# that's not working:
form some_form_config do
  toggle show_on: "some-event" do
    form_input key: :some_input_key, type: :text
  end
end
```

We're working on a better decoupling of the form child components. In the mean time you can enable dynamic child component rendering utilizing Vue.js directly. We will shortly publish a guide towards that topic.

## Examples

These examples show generic use cases and can be used as a guideline of what is possible with the form core component.

Beforehand, we define some example routes for the form input in our `config/routes.rb`:

```ruby
post '/success_form_test/:id', to: 'form_test#success_submit', as: 'success_form_test'
post '/failure_form_test/:id', to: 'form_test#failure_submit', as: 'failure_form_test'
post '/model_form_test', to: 'model_form_test#model_submit', as: 'model_form_test'
```

We also configure our example controllers to accept form input and react in a predictable and verbose way:

```ruby
class FormTestController < ApplicationController

  def success_submit
    render json: { message: 'server says: form submitted successfully' }, status: 200
  end

  def failure_submit
    render json: {
      message: 'server says: form had errors',
      errors: { foo: ['seems to be invalid'] }
    }, status: 400
  end

end
```

```ruby
class ModelFormTestController < ApplicationController

  def model_submit
    @test_model = TestModel.create(model_params)
    if @test_model.errors.any?
      render json: {
        message: 'server says: something went wrong!',
        errors: @test_model.errors
      }, status: :unprocessable_entity
    else
      render json: {
        message: 'server says: form submitted successfully!'
      }, status: :ok
    end
  end

  protected

  def model_params
    params.require(:test_model).permit(:title, :description, :status, some_data: [], more_data: [])
  end

end
```

### Example 1: Async submit request with clientside payload

On our example page, we define a form that accepts text input and has a submit button.

```ruby
class ExamplePage < Matestack::Ui::Page

  def response
    form form_config do
      form_input id: 'my-test-input', key: :foo, type: :text
      form_submit do
        button text: 'Submit me!'
      end
    end
  end

  def form_config
    {
      for: :my_object,
      method: :post,
      path: :success_form_test_path,
      params: {
        id: 42
      }
    }
  end

end
```

When we visit `localhost:3000/example`, fill in the input field with *bar* and click the submit button, our `FormTestController` receives the input.

Furthermore, our *bar* input disappears from the input field - Easy!

### Example 2: Async submit request with failure event

This time, we break the form input on purpose to test our failure message! Again, we define our example page. Notice that we explicitly aim for our `failure_form_test_path`.

```ruby
class ExamplePage < Matestack::Ui::Page

  def response
    form form_config do
      form_input id: 'my-test-input', key: :foo, type: :text
      form_submit do
        button text: 'Submit me!'
      end
    end
    toggle show_on: 'my_form_failure' do
      plain "{{event.data.message}}"
      plain "{{event.data.errors}}"
    end
  end

  def form_config
    {
      for: :my_object,
      method: :post,
      path: :failure_form_test_path,
      params: {
        id: 42
      },
      failure: {
        emit: 'my_form_failure'
      }
    }
  end

end
```

Now, when we visit our example page on `localhost:3000/example` and fill in the input field with e.g. *bar* and hit the submit button, we get displayed both `server says: form had errors` and `'foo': [ 'seems to be invalid' ]`. Just what we expected to receive!

### Example 3: Async submit request with success transition

In this example, things get a bit more complex. We now want to transition to another page of our application after successfully submitting a form!

In order to additionally show a success/failure message, we define our matestack app layout with messages, using the *async core component*:

```ruby
class ExampleApp < Matestack::Ui::App

  def response
    heading size: 1, text: 'My Example App Layout'
    main do
      yield_page
    end
    toggle show_on: 'my_form_success', hide_after: 300 do
      plain "{{event.data.message}}"
    end
    toggle show_on: 'my_form_failure', hide_after: 300 do
      plain "{{event.data.message}}"
      plain "{{event.data.errors}}"
    end
  end

end
```

On our first example page, we define our form to transfer us to the second page (`form_test_page_2_path`) on successful input:

```ruby
class ExampleApp::Pages::ExamplePage < Matestack::Ui::Page

  def response
    heading size: 2, text: 'This is Page 1'
    form form_config do
      form_input id: 'my-test-input-on-page-1', key: :foo, type: :text
      form_submit do
        button text: 'Submit me!'
      end
    end
  end

  def form_config
    {
      for: :my_object,
      method: :post,
      path: :success_form_test_path,
      params: {
        id: 42
      },
      success: {
        emit: 'my_form_success',
        transition: {
          path: :form_test_page_2_path,
          params: {
            id: 42
          }
        }
      }
    }
  end

end
```

On the second example page, we aim for our failure path (`failure_form_test_path`) on purpose and define our form to transfer us to the first page (`form_test_page_1_path`) on failed input:

```ruby
class ExampleApp::Pages::SecondExamplePage < Matestack::Ui::Page

  def response
    heading size: 2, text: 'This is Page 2'
    form form_config do
      form_input id: 'my-test-input-on-page-2', key: :foo, type: :text
      form_submit do
        button text: 'Submit me!'
      end
    end
  end

  def form_config
    {
      for: :my_object,
      method: :post,
      path: :failure_form_test_path,
      params: {
        id: 42
      },
      failure: {
        emit: 'my_form_failure',
        transition: {
          path: :form_test_page_1_path
        }
      }
    }
  end

end
```

Of course, to reach our two newly defined pages, we need to make them accessible through a controller:

```ruby
class ExampleAppPagesController < ExampleController
  include Matestack::Ui::Core::ApplicationHelper
  matestack_app ExampleApp

  def page1
    render ExampleApp::Pages::ExamplePage
  end

  def page2
    render ExampleApp::Pages::SecondExamplePage
  end

end
```

We also need to extend our routes in `config/routes.rb` to handle the new routes:

```ruby
scope :form_test do
  get 'page1', to: 'example_app_pages#page1', as: 'form_test_page_1'
  get 'page2/:id', to: 'example_app_pages#page2', as: 'form_test_page_2'
end
```

Now, if we visit `localhost:form_test/page1`, we can fill in the input field with e.g. *bar* and click the submit button.

We then get displayed our nice success message (`server says: form submitted successfully`) and get transferred to our second page.

If we fill in the the input field there and hit the submit button, we not only see the failure messages (`server says: form had errors` and `'foo': [ 'seems to be invalid' ]`), we also get transferred back to the first page, just the way we specified this behavior in the page definition above!

### Example 3.1: Async submit request with success transition - dynamically determined by server

In the example shown above, the `success` `transition` is statically defined. Sometimes the `transition` needs to be dynamically controlled within the server action.
Imagine creating a new Active Record instance with a `form`. If you want to show the fresh instance on another page and therefore want to define a `transition` after successful form submission, you would need to know the ID of the fresh instance! That is not possible, as the ID is auto-generated and depends on the current environment/state. Therefore you can tell the `form` component to follow a transition, which the server action defines after creating the new instance (and now knowing the ID):

On the `page`:
```ruby
#...

def form_config
  {
    for: :my_object,
    method: :post,
    path: :success_form_test_path,
    success: {
      emit: 'my_form_success',
      transition: {
        follow_response: true # follow the serverside transition
      }
    }
  }
end
```
On the `controller` `action`:

```ruby
#...
def model_submit
  @test_model = TestModel.create(model_params)
  if @test_model.errors.any?
    render json: {
      message: 'server says: something went wrong!',
      errors: @test_model.errors
    }, status: :unprocessable_entity
  else
    render json: {
      message: 'server says: form submitted successfully!',
      transition_to: some_other_path(id: @test_model.id) #tell the form component where to transition to with the id, which was not available before
    }, status: :ok
  end
end
```

### Example 4: Multiple input fields of different types

Of course, our input core component accepts not only 'text', but very different input types: In this example, we will introduce 'password', 'number', 'email', 'range' types!

On our example page, we define the input fields, together with a `type: X` configuration:

```ruby
class ExamplePage < Matestack::Ui::Page

  def response
    form form_config do
      form_input id: 'text-input',      key: :text_input, type: :text
      form_input id: 'email-input',     key: :email_input, type: :email
      form_input id: 'password-input',  key: :password_input, type: :password
      form_input id: 'number-input',    key: :number_input, type: :number
      form_input id: 'range-input',     key: :range_input, type: :range
      form_submit do
        button text: 'Submit me!'
      end
    end
  end

  def form_config
    {
      for: :my_object,
      method: :post,
      path: :success_form_test_path,
      params: {
        id: 42
      }
    }
  end

end
```

Now, we can visit `localhost:3000/example` and fill in the input fields with various data that then gets sent to the corresponding path, in our case `success_form_test_path`.

### Example 5: Initialization with a value

Our form_input field doesn't need to be empty when we load the page. We can `init` it with all kinds of values:

```ruby
class ExamplePage < Matestack::Ui::Page

  def response
    form form_config do
      form_input id: 'text-input', key: :text_input, type: :text, init: 'some value'
      form_submit do
        button text: 'Submit me!'
      end
    end
  end

  def form_config
    {
      for: :my_object,
      method: :post,
      path: :success_form_test_path,
      params: {
        id: 42
      }
    }
  end

end
```

Now, when we visit `localhost:3000/example`, we see our input field already welcomes us with the value *some value*!

### Example 6: Pre-filling the input field with a placeholder

Instead of a predefined value, we can also just show a placeholder in our form_input component:

```ruby
class ExamplePage < Matestack::Ui::Page

  def response
    form form_config do
      form_input id: 'text-input', key: :text_input, type: :text, placeholder: 'some placeholder'
      form_submit do
        button text: 'Submit me!'
      end
    end
  end

  def form_config
    {
      for: :my_object,
      method: :post,
      path: :success_form_test_path,
      params: {
        id: 42
      }
    }
  end

end
```

Now, when we visit `localhost:3000/example`, the input field is technically empty, but we see the text *some placeholder*. In contrary to the `init` value in example 5, the placeholder can't get submitted)!

### Example 7: Defining a label

Another useful feature is that we can also give a *label* to our form_input!

```ruby
class ExamplePage < Matestack::Ui::Page

  def response
    form form_config do
      form_input id: 'text-input', key: :text_input, type: :text, label: 'some label'
      form_submit do
        button text: 'Submit me!'
      end
    end
  end

  def form_config
    {
      for: :my_object,
      method: :post,
      path: :success_form_test_path,
      params: {
        id: 42
      }
    }
  end

end
```

Now, when we visit `localhost:3000/example`, the input field carries a *some label*-label.

### Example 8: Asynchronously display error messages

Here, we aim for the `failure_form_test_path` on purpose to check how error messages are handled!

```ruby
class ExamplePage < Matestack::Ui::Page

  def response
    form form_config do
      form_input id: 'text-input', key: :foo, type: :text
      form_submit do
        button text: 'Submit me!'
      end
    end
  end

  def form_config
    {
      for: :my_object,
      method: :post,
      path: :failure_form_test_path,
      params: {
        id: 42
      }
    }
  end

end
```

If we head to `localhost:3000/example`, and fill in the input field with, e.g., *text* and click the submit button, we will get displayed our error message of `seems to be invalid`. Neat!


### Example 9: Mapping the form to an Active Record Model

To test the mapping of our form to an Active Record Model, we make sure our `TestModel`'s description can't be empty:

```ruby
class TestModel < ApplicationRecord

  validates :description, presence:true

end
```

Now, on our example page, we _prepare_ a new instance of our `TestModel` that we then want to save through the form component:

```ruby
class ExamplePage < Matestack::Ui::Page

  def prepare
    @test_model = TestModel.new
    @test_model.title = 'Title'
  end

  def response
    form form_config do
      form_input id: 'title', key: :title, type: :text
      form_input id: 'description', key: :description, type: :text
      form_submit do
        button text: 'Submit me!'
      end
    end
  end

  def form_config
    {
      for: @test_model,
      method: :post,
      path: :model_form_test_path
    }
  end

end
```

Notice that we only _prepared_ the title, but missed out on the description.

If we head to our example page on `localhost:3000/example`, we can already see the title input field filled in with *Title*. Trying to submit the form right away gives us the error message (`can't be blank`) because the description is, of course, still missing!

After filling in the description with some input and hitting the submit button again, the instance of our `TestModel` gets successfully saved in the database - just the way we want it to work.
