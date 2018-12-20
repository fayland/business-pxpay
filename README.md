# SYNOPSIS

    use Business::PxPay;

    my $pxpay = Business::PxPay->new(
        userid => 'TestAccount',
        key    => 'c9fff215b9e2add78d252b78e214880b46a906e73190a380483c1c29acab4157'
    );

    # when submit the cart order
    if ( $submit_order ) {
        my $rtn = $pxpay->request($args); # $args from CGI params
        if ( exists $rtn->{valid} and $rtn->{valid} == 1 ) {
            print $q->redirect( $rtn->{URI} );
        } else {
            die Dumper(\$rtn);
        }
    }
    # when user returns back from pxpal
    elsif ( $in_return_or_cancel_page or $params->{result} ) {
        my $rtn = $pxpay->result($params->{result});
        if ( exists $rtn->{valid} and $rtn->{valid} == 1 ) {
            print "Transaction Success!\n";
        } else {
            print "Transaction Failed!\n";
        }
    }

# DESCRIPTION

PX Pay - Payment Express [http://www.paymentexpress.com/](http://www.paymentexpress.com/)

## new

    my $pxpay = Business::PxPay->new(
        userid => $user,
        key    => $key
    );

- `userid` (required)
- `key` (required)

    PxPayUserId & PxPayKey

- `ua`
- `ua_args`

    By default, we use LWP::UserAgent->new as the UserAgent. you can pass `ua` or `ua_args` to use a different one.

- `url`

        my $pxpay = Business::PxPay->new(
            userid => $user,
            key    => $key,
            url    => 'https://sec2.paymentexpress.com/pxpay/pxaccess.aspx', # to test?
        );

    The URL is 'https://www.paymentexpress.com/pxpay/pxaccess.aspx' by default.

## Arguments

All those arguments can be passed into Business::PxPay->new() or pass into $pxpay->request later

    {
        TxnType => 'Purchase',
        Amount  => 10.9,
        UrlFail => 'http://test.com',
        UrlSuccess => 'http://example.com',
        MerchantReference => 'Test Transaction',
        EmailAddress => 'test@example.com',
        TxnData1 => 'test=A',
        TxnData2 => 'data2=B',
        TxnData3 => 'data3=C',
    };

- `Amount` (required)

    Amount value in d.cc format

- `Currency` (required)

    [http://www.paymentexpress.com/technical\_resources/ecommerce\_hosted/pxpay.html#currencyinput](http://www.paymentexpress.com/technical_resources/ecommerce_hosted/pxpay.html#currencyinput)

- `UrlFail` (required)

    URL of the merchant transaction failure page. No parameters ("&" or "?") are permitted.

- `UrlSuccess` (required)

    URL of the merchant transaction success page. No parameters ("&" or "?") are permitted.

- `TxnType` (required)

    "Auth" or "Purchase"

- `MerchantReference` (required)

    Reference field to appear on transaction reports

- `BillingId`

    Optional identifier when adding a card for recurring billing

- `EmailAddress`

    Optional email address

- `EnableAddBillCard`

    Required when adding a card to the DPS system for recurring billing. Set element to "1" for true

- `TxnId`

    A value that uniquely identifies the transaction

- `TxnData1`
- `TxnData2`
- `TxnData3`

    Optional free text

- `Opt`

    Optional additional parameter

## request

    my $xml = $pxpay->request_xml($args);
    my $rtn = $pxpay->request($args);

request and parse the response XML into HASHREF. sample:

    $VAR1 = \{
        'URI' => 'https://sec2.paymentexpress.com/pxpay/pxpay.aspx?userid=
TestAccount&request=v51flwn7rvSNcbY86uRMdJ74XB2gHd8ZY-WHqyEYoPm9xd1ROXX00pXYkkuk
dleLlS402E65EjOSCkrqvmAsZUWRCck8RkmIJcRLvG0KZLi7PQRBfpIQ0wzKwdHGKvBCpqhRH6Tx-w93
MRYsP0ThOK4btgTneR_hGEk0supyLeE1taNWCkyFj8KX7rzZ9ncdWRlmciNBsiV4zX4DQ_7Poi9qiblI
5o0Gm49yb90kUlUtH1hrV3ulzidQbn0CcQKhHFKGX8IVMXiAtVN29r_Cgdzc7dOrwOxY-LBY2h4Or5GQ
hJHB96kjBziu3GyGBvaGfsosNodT3-wyM29A5M-Z62ITkno6JUA6H4',
        'valid' => '1'
    };

Usually you need redirect to the $rtn->{URI} when valid is 1

## result

    my $xml = $pxpay->result_xml($ResponseCode);
    my $rtn = $pxpay->result($ResponseCode);
    if ( exists $rtn->{valid} and $rtn->{valid} == 1 ) {
        print "Transaction Success!\n";
    } else {
        print "Transaction Failed!\n";
    }

PxPay will POST to your `UrlSuccess` (or `UrlFail`) when you finish the transaction (or click Cancel button). the POST would contain a param **result** which you can request to get the transaction status.

## TIPS

### I need params in `UrlSuccess`

For example, you want your UrlSuccess to be 'http://mysite.com/cgi-bin/cart.cgi?cart\_id=ABC'.

you need write the request like:

    my $rtn = $pxpay->request(
        # others
        UrlSuccess => 'http://mysite.com/cgi-bin/cart.cgi',
        TxnData1 => 'ABC',
    );

and you can get the `TxnData1` in

    my $rtn = $pxpay->result($ResponseCode);
    my $cart_id = $rtn->{TxnData1}
