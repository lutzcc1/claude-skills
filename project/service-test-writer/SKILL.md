---
name: service-test-writer
description: Writes comprehensive service tests for Sprout Rails project following the project's testing patterns and conventions. Use this when the user asks to write tests for a service, or when you've just created or modified a service.
---

# Service Test Writer

This skill helps write comprehensive service tests following Sprout's testing conventions and patterns.

## Instructions

Follow these steps in order:

### 1. Understand the Service

Before writing tests, analyze the service implementation:

1. **Read the service file** - Understand the service's purpose, arguments, and return values
2. **Identify the service's responsibilities** - What business logic does it implement?
3. **Identify dependencies** - What models, external APIs, or other services does it use?
4. **Identify edge cases** - What validations, error conditions, and edge cases exist?
5. **Note any state changes** - What database records are created, updated, or destroyed?

**IMPORTANT**: Only test core business logic. Do NOT test:

- Attribute accessors or simple getters/setters
- Inheritance checks
- Argument handling unless it's a critical validation
- Environment-specific behavior

### 2. Plan Test Coverage

Create a mental model or outline of test scenarios:

1. **Happy path tests** - Service succeeds under normal conditions
2. **Validation tests** - Service fails with missing or invalid arguments
3. **Edge case tests** - Boundary conditions, empty states, etc.
4. **Error handling tests** - External API failures, database errors, timeouts
5. **State change tests** - Database records are created/updated correctly
6. **Transaction rollback tests** - Failures properly rollback database changes
7. **Integration tests** - External service calls (Stripe, Depot, etc.) are made correctly

### 3. Identify Required Seeds

**CRITICAL**: Always use Oaken seeds for test data. Do NOT create/update records in tests unless absolutely necessary.

1. **Find existing seeds** - Search for seeds in `test/seeds/cases/`
2. **Use standard seeds** - Prefer standard seeds like:
   - `cases/accounts/standard/default`
   - `cases/clusters/standard/default`
   - `cases/infrastructure/omc_vault/default`
   - `cases/subscriptions/standard/active`
   - `cases/subscriptions/standard/canceled`
   - `cases/subscriptions/standard/past_due`
3. **AVOID** - `cases/accounts/standard_lies` (deprecated)
4. **Create new seeds if needed** - If existing seeds don't cover your use case, create new ones

### 4. Write the Test File

#### File Structure

```ruby
require "test_seeds_helper"

# Include helpers as needed
# include StripeTestHelper for Stripe API stubbing
# include FeatureFlipperHelper for feature flags

class ServiceNameTest < ActiveSupport::TestCase
  # For Stripe integration tests
  # include StripeTestHelper

  setup do
    # Load Oaken seeds
    seed "cases/accounts/standard/default"
    seed "cases/clusters/standard/default"
    # ... other seeds as needed

    # Initialize instance variables from seeds
    @cluster = clusters.standard
    @account = accounts.standard

    # Set up any stubs for external APIs
    # stub_stripe_customer_create
    # stub_request(:get, "https://api.example.com/...").to_return(...)
  end

  test "description of what this test does" do
    # Call the service
    result = ServiceName.call(@cluster, other_args)

    # Assert the result status
    assert result.success?
    # or: assert result.failure?
    # or: assert result.retry?

    # Assert state changes (reload records first!)
    # Reloading must always happen in its own line
    @cluster.reload

    assert_equal expected_value, @cluster.some_attribute

    # Assert external API calls were made
    # assert_requested :post, "https://api.stripe.com/v1/...", times: 1
  end
end
```

#### Multiple Test Classes Pattern

When testing different scenarios (like different subscription states), create separate test classes:

```ruby
class ServiceNameWithNoSubscriptionTest < ActiveSupport::TestCase
  # Tests for accounts without subscriptions
end

class ServiceNameWithActiveSubscriptionTest < ActiveSupport::TestCase
  # Tests for accounts with active subscriptions
end

class ServiceNameWithCanceledSubscriptionTest < ActiveSupport::TestCase
  # Tests for accounts with canceled subscriptions
end

class ServiceNameTransactionTest < ActiveSupport::TestCase
  # Tests for transaction rollbacks
end
```

### 5. Test Patterns

#### Testing Service Success

```ruby
test "successfully processes the cluster" do
  result = ServiceName.call(@cluster)

  assert result.success?
  # Access result data via reader attributes
  assert_equal expected_value, result.some_attribute
end
```

#### Testing Service Failure

```ruby
test "returns failure when cluster is missing" do
  result = ServiceName.call(nil)

  assert result.failure?
  assert result.error.present?
  assert_includes result.error, "expected error message fragment"
end
```

#### Testing Service Retry

```ruby
test "returns retry signal on timeout" do
  # Stub to raise timeout error
  SomeExternalService.stubs(:call).raises(Net::TimeoutError.new("timeout"))

  result = ServiceName.call(@cluster)

  assert result.retry?
  assert result.retry_options.present?
end
```

#### Testing Validation Errors

```ruby
test "returns error if arguments are blank" do
  [
    [nil, nil],
    [@cluster, nil],
    [nil, @start_time]
  ].each do |args|
    result = ServiceName.call(*args)
    assert result.failure?
    assert result.error.present?
  end
end
```

#### Testing State Changes

```ruby
test "creates a new record" do
  refute @cluster.some_association.present?

  result = ServiceName.call(@cluster)

  @cluster.reload

  assert result.success?
  assert @cluster.some_association.present?
  assert_equal expected_value, @cluster.some_association.attribute
end
```

#### Testing Transaction Rollbacks

```ruby
test "transaction rollback when something fails" do
  refute @cluster.some_association.present?

  # Stub to cause failure
  SomeModel.any_instance.stubs(:save!).raises(ActiveRecord::RecordInvalid.new(SomeModel.new))

  result = ServiceName.call(@cluster)

  @cluster.reload

  assert result.failure?
  assert result.error.present?
  refute @cluster.some_association.present?
end
```

#### Testing External API Integration (Stripe Example)

```ruby
test "creates Stripe customer" do
  expected_stripe_id = "cus_123456789"
  stub_stripe_customer_create(body: {id: expected_stripe_id})

  result = ServiceName.call(@account)

  @account.reload

  assert result.success?
  assert_requested :post, "https://api.stripe.com/v1/customers", times: 1
  assert_equal expected_stripe_id, @account.stripe_id
end
```

#### Testing with Webmock (Generic HTTP)

```ruby
test "fetches data from external API" do
  stub_request(:get, "https://api.example.com/data")
    .to_return(status: 200, body: {result: "success"}.to_json)

  result = ServiceName.call(@cluster)

  assert result.success?
  assert_requested :get, "https://api.example.com/data", times: 1
end
```

#### Testing with Mocha (Method Stubbing)

```ruby
test "handles private routing correctly" do
  @cluster.pod.stubs(:private_routing?).returns(true)

  result = ServiceName.call(@cluster)

  assert result.success?
end
```

#### Testing Request Bodies

```ruby
test "sends correct data to Stripe" do
  expected_body_keys = [
    "customer",
    "collection_method",
    "metadata[slug]",
    "items[0][price]"
  ]

  result = ServiceName.call(@cluster)

  assert result.success?
  assert_requested :post, "https://api.stripe.com/v1/subscriptions" do |req|
    form_request_body = URI.decode_www_form(req.body).to_h
    assert_equal expected_body_keys, form_request_body.keys
  end
end
```

### 6. Common Test Helpers

#### Available Test Helpers

- `step_through_all_states(action)` - Iterate through state machine steps until terminal state
- `stub_features(feature_name: true)` - Stub feature flags
- `fixture_file("path/to/fixture.json")` - Load fixture files from `test/stubs/`
- `stub_amplitude_api` - Stub Amplitude analytics API (automatically called in integration tests)
- `regexp_for_json_field(json)` - Create regex matcher for partial JSON matching in Webmock

#### Stripe Test Helper Methods

When including `StripeTestHelper`, you have access to:

- `stub_stripe_customer_create(body: {}, status: 200)`
- `stub_stripe_customer_get(body: {}, status: 200)`
- `stub_stripe_subscription_create(body: {}, status: 200) { |request, body| ... }`
- `stub_stripe_subscription_get(body: {}, status: 200)`
- `stub_stripe_subscription_update(body: {}, status: 200)`
- `stub_stripe_subscription_item_create(body: {}, status: 200)`
- `stub_stripe_subscription_item_update(body: {}, status: 200)`
- `stub_stripe_subscription_item_delete(body: {}, status: 200)`
- `stub_stripe_payment_intent_success`
- And many more... (see `test/support/stripe_test_helper.rb`)

### 7. Running Tests

After writing tests:

```bash
# Run specific test file
rails test test/with_seeds/services/service_name_test.rb

# Run specific test
rails test test/with_seeds/services/service_name_test.rb:42

# Run all service tests
rails test test/with_seeds/services/
```

### 8. Quality Checklist

Before considering the test complete:

- [ ] Tests focus on core business logic only
- [ ] All test data comes from Oaken seeds (no manual record creation unless justified)
- [ ] Tests have descriptive names explaining what they test
- [ ] Happy path is tested
- [ ] Validation/error cases are tested
- [ ] External API calls are stubbed and verified
- [ ] Database state changes are tested with `reload`
- [ ] Transaction rollbacks are tested for failure scenarios
- [ ] Tests follow the existing patterns in the codebase
- [ ] Tests pass when run individually and in the full suite

## Example Test File

Here's a complete example following all conventions:

```ruby
require "test_seeds_helper"

class ClusterCredentialCreatorTest < ActiveSupport::TestCase
  setup do
    seed "cases/accounts/standard/default"
    seed "cases/clusters/standard/default"

    @cluster = clusters.standard
  end

  test "it inherits from ApplicationService" do
    assert_includes ClusterCredentialCreator.ancestors, ApplicationService
  end

  test "successfully creates credentials" do
    refute @cluster.credentials.present?

    result = ClusterCredentialCreator.call(@cluster, username: "admin", password: "secret")

    @cluster.reload

    assert result.success?
    assert @cluster.credentials.present?
    assert_equal "admin", @cluster.credentials.username
  end

  test "returns error when cluster is nil" do
    result = ClusterCredentialCreator.call(nil, username: "admin", password: "secret")

    assert result.failure?
    assert result.error.present?
  end
end

class ClusterCredentialCreatorTransactionTest < ActiveSupport::TestCase
  setup do
    seed "cases/accounts/standard/default"
    seed "cases/clusters/standard/default"

    @cluster = clusters.standard
  end

  test "transaction rollback when credential save fails" do
    refute @cluster.credentials.present?

    Credential.any_instance.stubs(:save!).raises(ActiveRecord::RecordInvalid.new(Credential.new))

    result = ClusterCredentialCreator.call(@cluster, username: "admin", password: "secret")

    @cluster.reload

    assert result.failure?
    refute @cluster.credentials.present?
  end
end
```

## Error Handling

- If seeds don't exist, search for them first with `find test/seeds -name "*.yml" | grep pattern`
- If Stripe stubs don't exist in `StripeTestHelper`, add them following existing patterns
- If tests fail, ensure you're reloading records before assertions
- If parallel tests fail, ensure seeds are unique per test class
- If external API stubs don't match, check the request URL pattern in the stub

## Notes

- This skill is focused on service tests only (not controllers, models, or mailers)
- Always verify the service contract: `Service::Success`, `Service::Failure`, or `Service::Retry`
- Be critical: Don't test trivial things, focus on business logic
- When in doubt about seeds, search existing test files for similar scenarios
- Remember: Services should have reader attributes for accessing result data
