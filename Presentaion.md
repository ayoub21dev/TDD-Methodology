# TDD Methodology

## C6 Competency Completion

- Project: Laravel Unit Testing
- Task: Live Coding
- Status: To Do
- Topic: TDD (Test-Driven Development) Methodology

---

## Objective

This presentation demonstrates how TDD works in a Laravel project by showing:

- how to start with a failing test
- how to move from Red to Green
- how to write a complete test using only an interface contract
- how implementation comes after the test, not before

---

## What Is TDD?

TDD is a development methodology where the test is written before the production code.

The workflow is:

1. Write a test for a behavior that does not exist yet
2. Run the test and confirm it fails
3. Write the minimum code needed to make it pass
4. Refactor while keeping the test green

---

## The TDD Cycle

### Red

- Write a new test
- The test fails because the feature is missing

### Green

- Add the smallest implementation possible
- The test passes

### Refactor

- Improve naming, structure, or duplication
- Keep behavior unchanged

---

## Why TDD In Laravel?

- It gives fast feedback during development
- It improves design before adding too much code
- It helps isolate business logic from framework details
- It makes regression detection easier
- It fits well with PHPUnit, Pest, Mockery, and Laravel's container

---

## Live Coding Scenario

We want to build a small service that converts an amount from one currency to another.

Important constraint:

- we only know the contract
- the concrete implementation of that contract does not exist yet

Contract-first design:

```php
<?php

namespace App\Contracts;

interface CurrencyRateProviderInterface
{
    public function rate(string $from, string $to): float;
}
```

At this point, there is no concrete class yet.

---

## Red: Write The Test First

The test describes the expected behavior before the real implementation exists.

```php
<?php

namespace Tests\Unit;

use App\Actions\ConvertCurrencyAction;
use App\Contracts\CurrencyRateProviderInterface;
use Mockery;
use PHPUnit\Framework\TestCase;

class ConvertCurrencyActionTest extends TestCase
{
    protected function tearDown(): void
    {
        Mockery::close();
        parent::tearDown();
    }

    public function test_it_converts_an_amount_using_the_rate_provider_contract(): void
    {
        $provider = Mockery::mock(CurrencyRateProviderInterface::class);

        $provider
            ->shouldReceive('rate')
            ->once()
            ->with('USD', 'MAD')
            ->andReturn(10.0);

        $action = new ConvertCurrencyAction($provider);

        $result = $action->execute(100.0, 'USD', 'MAD');

        $this->assertSame(1000.0, $result);
    }
}
```

---

## Why Is This Test Red?

It fails because:

- `ConvertCurrencyAction` does not exist yet
- the behavior is specified, but the production code is missing

This is the expected TDD starting point.

Red means the test is useful because it detects missing functionality.

---

## Green: Add The Minimum Implementation

Now we write only enough code to satisfy the test.

```php
<?php

namespace App\Actions;

use App\Contracts\CurrencyRateProviderInterface;

class ConvertCurrencyAction
{
    public function __construct(
        private CurrencyRateProviderInterface $provider
    ) {
    }

    public function execute(float $amount, string $from, string $to): float
    {
        return $amount * $this->provider->rate($from, $to);
    }
}
```

Now the test passes.

This is the Green phase.

---

## Refactor

Once the test is green, we can improve the code safely.

Possible refactors:

- round the result to 2 decimals
- validate negative amounts
- rename classes for domain clarity
- extract value objects later if needed

Example:

```php
public function execute(float $amount, string $from, string $to): float
{
    return round($amount * $this->provider->rate($from, $to), 2);
}
```

The important rule:

Refactor only while tests stay green.

---

## What This Demonstrates

This example respects TDD because:

- the behavior is defined before implementation
- the first visible result is a failing test
- the code moves from Failure to Success
- the class depends on an interface, not a concrete service
- the interface can exist without any real provider implementation

This is good design because it keeps the code testable and loosely coupled.

---

## Laravel Testing Commands

Create the test:

```bash
php artisan make:test ConvertCurrencyActionTest --unit
```

Run the tests:

```bash
php artisan test
```

Or run only one test file:

```bash
php artisan test tests/Unit/ConvertCurrencyActionTest.php
```

---

## Expected Progression During Demo

1. Create the interface contract
2. Write the unit test
3. Run the test and observe failure
4. Create `ConvertCurrencyAction`
5. Run the test again and observe success
6. Refactor carefully
7. Run the test suite again

---

## Key Takeaways

- TDD starts with a failing test
- the test defines the required behavior
- implementation is added only after the failure is confirmed
- interfaces improve flexibility and testability
- Laravel supports this workflow very well through PHPUnit and dependency injection

---

## Conclusion

TDD is not only a testing technique. It is a way to design software step by step.

For this Laravel unit testing project, the main goal is to clearly show the path:

Failure -> Green -> Refactor

That progression proves the feature was driven by the test and not by guesswork.
