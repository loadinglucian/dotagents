# PHP Development Reference

Load this reference when the task needs the exact compatibility examples and detailed pattern snippets inherited from the original `php` and `pest` skills.

## PHP Examples

### Example: Yoda Conditions (Correct)

```php
if (null === $value) { ... }
if ('' === trim($name)) { ... }
if (0 === count($items)) { ... }
```

### Example: Yoda Conditions (Wrong)

```php
if ($value === null) { ... }  // constant should be on LEFT
if (trim($name) === '') { ... }  // literal should be on LEFT
```

### Example: Two Variables

```php
// Two variables - Yoda doesn't apply
if ($typedName !== $server->name) { ... }
```

### Example: PHPStan Annotation (Correct)

```php
/** @var string $apiToken */
$apiToken = $this->env->get(['API_TOKEN']);
```

### Example: PHPStan Annotation (Wrong)

```php
assert(is_string($apiToken));  // don't use assert() in production
```

### Example: Comment Structure

```php
// ----
// Section Header (h1)
// ----

//
// Subsection (h2)
// ----

//
// Minor heading (h3)

// Regular comment (p)
```

## Pest Examples

### Example: File Structure

```php
<?php

declare(strict_types=1);

//
// {Feature} tests
// ----

it('does something specific', function () {
    // ARRANGE
    $service = new Service(mock(Dependency::class));

    // ACT
    $result = $service->action();

    // ASSERT
    expect($result)->toBe('expected');
});

it('throws when invalid input', function () {
    $service = new Service();

    // ACT & ASSERT
    expect(fn () => $service->action('invalid'))
        ->toThrow(InvalidArgumentException::class, 'Expected message');
});
```

### Example: Exception Test

```php
it('throws on negative price', function () {
    $calculator = new PriceCalculator();

    // ACT & ASSERT
    expect(fn () => $calculator->calculateTotal([['price' => -10]]))
        ->toThrow(InvalidArgumentException::class);
});
```

### Example: Naming (Correct)

```php
it('returns empty array when no servers configured')
it('throws when SSH connection fails')
it('creates deploy key with correct permissions')
```

### Example: Naming (Wrong)

```php
it('test1')
it('works')
it('should return correct value')
```

### Example: Unit Test DI

```php
it('parses config correctly', function () {
    $mockFs = mock(FilesystemInterface::class);
    $mockFs->shouldReceive('read')->with('/path')->andReturn('content');

    $service = new ConfigService($mockFs);
    $result = $service->parse('/path');

    expect($result)->toBe(['key' => 'value']);
});
```

### Example: Command Test DI

```php
it('adds server successfully', function () {
    $mockSSH = mock(SSHService::class);
    $mockSSH->shouldReceive('connect')->andReturn(true);

    $container = mockCommandContainer(ssh: $mockSSH);
    $command = $container->build(ServerAddCommand::class);

    // Test command execution
});
```

### Example: Dataset-Driven Scenarios

```php
it('validates server names', function (string $name, bool $valid) {
    $validator = new ServerValidator();

    expect($validator->isValidName($name))->toBe($valid);
})->with([
    'valid simple' => ['web-server', true],
    'valid with numbers' => ['web1', true],
    'invalid spaces' => ['web server', false],
    'invalid special chars' => ['web@server', false],
    'empty string' => ['', false],
]);
```

### Example: Assertion Chaining (Correct)

```php
expect($server)
    ->name->toBe('web1')
    ->and($server)
    ->host->toBe('192.168.1.1')
    ->and($server)
    ->port->toBe(22);
```

### Example: Assertion Chaining (Wrong)

```php
expect($server->name)->toBe('web1');
expect($server->host)->toBe('192.168.1.1');
expect($server->port)->toBe(22);
```

### Example: Mocking

```php
it('calls external service', function () {
    $mock = mock(ExternalService::class);
    $mock->shouldReceive('fetch')
        ->once()
        ->with('param')
        ->andReturn(['data']);

    $service = new MyService($mock);
    $result = $service->process();

    expect($result)->toBe('processed');
});
```

### Example: Arch Test

```php
arch('commands extend BaseCommand', function () {
    expect('Deployer\\Console\\')
        ->classes()
        ->toHaveSuffix('Command')
        ->toExtend(BaseCommand::class);
});

arch('services are final', function () {
    expect('Deployer\\Service\\')
        ->classes()
        ->toBeFinal();
});
```

## Pattern Snippets

### Forbidden Patterns

```php
// Type-only checks (prove nothing about behavior)
expect($x)->toBeInstanceOf(Class::class);
expect($x)->toBeArray();
expect($x)->not->toBeNull();

// Literally meaningless
expect(true)->toBeTrue();

// Time-dependent (test logic, not time)
sleep(1);
usleep(1000);
```

### Required Patterns

```php
// Verify actual values
expect($config->getValue('host'))->toBe('example.com');

// Verify mock interactions
$mock->shouldReceive('method')->with('param')->andReturn('result');

// Polling/timeout: use zero intervals
$service->waitForReady('id', timeout: 10, pollInterval: 0);
```

### Mock Patterns

```php
$mock->shouldReceive('method')->andReturn('value');
$mock->shouldReceive('method')->andReturn('first', 'second', 'third');
$mock->shouldReceive('method')->andThrow(new RuntimeException('error'));
$mock->shouldReceive('method')->once();
$mock->shouldReceive('method')->twice();
$mock->shouldReceive('method')->times(3);
$mock->shouldReceive('method')->never();
$mock->shouldReceive('method')->with('exact');
$mock->shouldReceive('method')->with(Mockery::any());
$mock->shouldReceive('method')->with(Mockery::type('string'));
```
