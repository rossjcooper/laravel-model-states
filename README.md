# WIP states for Laravel

[![Latest Version on Packagist](https://img.shields.io/packagist/v/spatie/state.svg?style=flat-square)](https://packagist.org/packages/spatie/:package_name)
[![Build Status](https://img.shields.io/travis/spatie/state/master.svg?style=flat-square)](https://travis-ci.org/spatie/:package_name)
[![Quality Score](https://img.shields.io/scrutinizer/g/spatie/state.svg?style=flat-square)](https://scrutinizer-ci.com/g/spatie/:package_name)
[![Total Downloads](https://img.shields.io/packagist/dt/spatie/state.svg?style=flat-square)](https://packagist.org/packages/spatie/:package_name)

## Installation

You can install the package via composer:

```bash
composer require spatie/laravel-state
```

## Usage

**Note**: make sure you're familiar with the basics of the [state pattern](https://en.wikipedia.org/wiki/State_pattern) before using this package.

This package adds state support to your Laravel models. First you'll have to use the `Spatie\State\HasStates` trait in your model. Now you're able to define state fields. 

State fields are simple properties on your model class, with a `@var` docblock. A field will be registered as a state field when its type extends the `Spatie\State\State` class.

When PHP 7.4 arrives, we'll add support for typed properties as well.

Here's an example of a `Payment` class with one state field, simply called `state`.

```php
use Spatie\State\HasStates;

class Payment extends Model
{
    use HasStates;

    /** @var \App\States\PaymentState */
    public $state;

    public function __construct(array $attributes = [])
    {
        parent::__construct($attributes);

        // Make sure to set the default state in the contructor
        $this->state = new Pending($this);
    }
}
```

Say there are be three possible payment states: pending, paid and failed. You'd be best to make one abstract base class for all payment states, and let concrete implementations extend it. Each concrete implementation can provide state-specific behaviour, as described by the [state pattern](https://en.wikipedia.org/wiki/State_pattern). 

```php
use Spatie\State\State;

// Concrete payment state classes can extend this base state
abstract class PaymentState extends State
{
    protected $payment;

    public function __construct(Payment $payment)
    {
        $this->payment = $payment;
    }

    abstract public function color(): string;
}
```

Now you can use the `state` field on your model directly as a `PaymentState` object, it will be properly saved and loaded to and from the database behind the scenes.

```php
$payment = Payment::create();

// Color depending on the current state
echo $payment->state->color();
```

Next up, you can make transition classes which will take care of state transitions for you. Here's an example of a transition class which will mark the payment as failed with an error message.

```php
use Spatie\State\Transition;

class PendingToFailed extends Transition
{
    private $message;

    public function __construct(string $message)
    {
        $this->message = $message;
    }

    public function __invoke(Payment $payment): Payment
    {
        // Only payments which state currently is pending can be handled by this transition
        $this->ensureInitialState($payment, Pending::class);

        $payment->state = new Failed($payment);
        $payment->failed_at = time();
        $payment->error_message = $this->message;

        $payment->save();

        return $payment;
    }
}
```

This transition is used like so:

```php
$pendingToFailed = new PendingToFailed('Error message from payment provider');

$payment = $pendingToFailed($payment);
```

### Testing

``` bash
composer test
```

### Changelog

Please see [CHANGELOG](CHANGELOG.md) for more information on what has changed recently.

## Contributing

Please see [CONTRIBUTING](CONTRIBUTING.md) for details.

### Security

If you discover any security related issues, please email freek@spatie.be instead of using the issue tracker.

## Postcardware

You're free to use this package, but if it makes it to your production environment we highly appreciate you sending us a postcard from your hometown, mentioning which of our package(s) you are using.

Our address is: Spatie, Samberstraat 69D, 2060 Antwerp, Belgium.

We publish all received postcards [on our company website](https://spatie.be/en/opensource/postcards).

## Credits

- [Brent Roose](https://github.com/brendt)
- [All Contributors](../../contributors)

## Support us

Spatie is a webdesign agency based in Antwerp, Belgium. You'll find an overview of all our open source projects [on our website](https://spatie.be/opensource).

Does your business depend on our contributions? Reach out and support us on [Patreon](https://www.patreon.com/spatie). 
All pledges will be dedicated to allocating workforce on maintenance and new awesome stuff.

## License

The MIT License (MIT). Please see [License File](LICENSE.md) for more information.