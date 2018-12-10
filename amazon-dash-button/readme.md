# Amazon Dash Button Example

This example illustrate how a workflow of a Amazon Dash Button can be implemented.

## Requirements

* Docker 18.09
* curl
* An account on [Zenaton](https://zenaton.com)

## Running the example

Build all docker images:

```sh
docker-compose build
```

Copy the `.env.example` to `.env`

```sh
cp .env.example .env
```

Retrieve your App ID and Api Token from the [Zenaton Website](https://zenaton.com/app/api) and use them to fill the `ZENATON_APP_ID` and `ZENATON_API_TOKEN` in the `.env` file.

Now we can install required dependencies using composer (inside docker of course!)

```sh
docker run --rm -v $(pwd):/application -w /application composer:1.6 composer install
```

You can now start your docker containers!

```sh
docker-compose up -d
```

Migrate the database and seed it:

```sh
docker-compose exec php-fpm php artisan migrate --seed
```

## Push the button

We provide a virtual Amazon Dash button. If you want to press it you can run the following command:

```sh
./dash.sh
```

At this point, your first order has been accepted. You can go to `http://localhost:1080/` and you should see an invoice in your mailbox.
You can run `./dash.sh` again to receive more invoices.

Now, go to `app/Services/Payment.php` and change method `chargeCustomerForOrder` to return false:

```php
$result = false;
```

Run the `./dash.sh` script again to make a new order. This time, you received an email with a link to enter new payment information.
When clicking it, you will see a form to enter payment details. Just enter a credit card number ending with an even digit to make the payment
a success. You will then receive an invoice in the mailbox, meaning the order was a success.

If you want to try the case when you don't provide new payment information, go to `app/Workflows/OrderFromDashButton.php` and change the wait
instruction from 14 days to 14 seconds.

```php
$event = (new Wait(OrderPaid::class))->seconds(14)->execute();
```

Now run the `./dash.sh` script one more time, and this time your order will be canceled. You can see the order being canceled in the log file
of the application: `storage/logs/laravel.log`. Everything happening is logged in this file!

## Uninstalling the project

Just run this command:

```sh
docker-compose down --volumes
```

After that, you're free to delete the project directory.
