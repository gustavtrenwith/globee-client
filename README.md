# GloBee PHP Client
This is a lightweight library to integrate your website with your GloBee account and accept crypto payments on 
your website.

## Server Requirements
The following packages need to be installed on the server:
```bash
php 7.1
php7.1-bcmath
```

## Installation with Composer
Run the following command in your project to add this package:
```bash
composer require gustavtrenwith/globee_client
```
Then run `composer update`.

## Environment Setup
You need to add the following to your .env file. 
```
ECDSA_SIN=
ECDSA_PRIVATE_KEY_HEX=
ECDSA_PUBLIC_KEY_COMPRESSED=
```

## Generate ECDSA Keypair Values
You need to use the following code to create the ECDSA Keypair Values. Once done, this code can safely be removed.
```php
GloBeeClient::generateECDSAKeys();
```
An associative array will be returned upon success:
```php
Array
(
  [private_key_hex] => 7a4fbece43963538cb8f9149b094906168d71be36cfb405e6930fddb42da2c7d
  [private_key_dec] => 55323065337948610870652254548527896513063178460294714145329611159...
  [public_key] => 043fbbf44c3da3fec12bf7bac254fd176adc3eaed79470932b574d8d60728eb206fb7a...
  [public_key_compressed] => 033fbbf44c3da3fec12bf7bac254fd176adc3eaed79470932b574d8d607...
  [public_key_x] => 3fbbf44c3da3fec12bf7bac254fd176adc3eaed79470932b574d8d60728eb206
  [public_key_y] => fb7ac7ac6959f75a6859a1a8d745db7e825a3c5c826e5b2e4950892b35772313
)
```
From the above array:
 - Store the `private_key_hex` as the value for the `ECDSA_PRIVATE_KEY_HEX` key in the environment variable file.
 - Store the `public_key_compressed` as the value for the `ECDSA_PUBLIC_KEY_COMPRESSED` key in the environment variable file.

## Generate the ECDSA Signature
You need to use the following code to create the ECDSA Keypair Values. Once done, this code can safely be removed.
```php
GloBeeClient::generateSignature(env('ECDSA_PUBLIC_KEY_COMPRESSED'));
```
This will return the BASE-58 encoded value beginning with the letter 'T' (specific value to SINs):
```
Tf61EPoJDSjbp6tGoyjbTKq7XLABPVcyUwY
```
- Store this string as the value for the `ECDSA_SIN` key in the environment variable file.

## Authenticate with GloBee

The easiest way to now link your website with GloBee, is by completing the following steps:
1) Using an app such as POSTMAN, send a `POST` request to `globee.com/tokens` with the following query parameters:
- label (e.g. My GloBee Client)
- id (The ECDSA_SIN value generated in the previous step)
- facade (e.g. merchant)
This will respond with a new token that will include a `pairingCode`.
2) Now go to the following URL, replacing `<pairingcode_goes_here>` with the actual pairing code:
`globee.com/api-access-request?pairingCode=<pairingcode_goes_here>` and validate the code.

You should now be linked to GloBee and able to start generating payment interstitials by following the steps in the usage example below.

## Usage Example
To create an invoice on GloBee and receive a redirect to a payment interstitial, you can start by modifying the below code:
```php
    $response = json_decode(
        GloBeeClient::createInvoice([
            'price' => (float) $amount,
            'currency_code' => 'USD',
            'guid' => 'Unique Identifier',
            'order_id' => 'Unique Order ID',
            'description' => 'Order Description',
            'notification_email' => 'Email address for payment notification (Client or Website Owner)',
            'notification_url' => 'ITN Callback URL',
            'redirect_url' => 'URL To redirect user to after payment',
            'passthrough_data' => 'Data you want to pass through to GloBee and back, to use on the ITN page',
            'transaction_speed' => 'high',
            'buyer_id' => 'Buyer ID',
            'buyer_name' => 'Buyer Name',
            'buyer_email' => 'Buyer Email',
        ])
    );
    if ( ! isset($response->success) || $response->success === false) {
        # Invalid Response Received
    }

    if ( ! isset($response->redirect_url)) {
        # No Redirect URL returned
    }

    # Redirect User to the redirect URL
    return redirect($response->redirect_url);
```

### Getting the passthrough data on the ITN page
To get the passthrough data on the ITN callback page you can use the following line, and then process the payment:
```php
    # Get the passthrough data sent to the notification page
    $data = GloBeeClient::getPosData($request);
    
    # Identify the payment from the $data and process it
```


## Feedback
For any questions or suggestions, feel free to contact me on `gtrenwith@gmail.com`
