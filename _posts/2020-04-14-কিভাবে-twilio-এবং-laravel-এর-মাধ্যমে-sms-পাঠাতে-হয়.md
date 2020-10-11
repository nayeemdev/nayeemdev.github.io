---
layout: post
title:  "কিভাবে Twilio এবং Laravel এর মাধ্যমে sms পাঠাতে হয়"
date:   2020-04-14
excerpt: "Laravel, Twilio"
tag:
- laravel 
- twilio
- sms
comments: true
---

![Twilio with Laravel](https://miro.medium.com/max/700/1*izwskFBcq6OUd2fHNhio1A.png)    

* GoTo this [Link](https://medium.com/@nayeemdev/how-to-send-sms-with-twilio-and-laravel-%E0%A6%95%E0%A6%BF%E0%A6%AD%E0%A6%BE%E0%A6%AC%E0%A7%87-twilio-%E0%A6%8F%E0%A6%AC%E0%A6%82-laravel-%E0%A6%8F%E0%A6%B0-%E0%A6%AE%E0%A6%BE%E0%A6%A7%E0%A7%8D%E0%A6%AF%E0%A6%AE%E0%A7%87-sms-%E0%A6%AA%E0%A6%BE%E0%A6%A0%E0%A6%BE%E0%A6%A4%E0%A7%87-%E0%A6%B9%E0%A7%9F-5330742cc3fe) for read it at medimum

* Dont forget to clap and response to this.
 


How to Send SMS with Twilio and Laravel || কিভাবে Twilio এবং Laravel এর মাধ্যমে SMS পাঠানো যায়
==============================================================================================

[![Nayeem Hossain](https://miro.medium.com/fit/c/96/96/1*JGSU3lXhMgW3RE0lKKu9LQ.jpeg)](https://medium.com/@nayeemdev?source=post_page-----5330742cc3fe--------------------------------)[Nayeem Hossain](https://medium.com/@nayeemdev?source=post_page-----5330742cc3fe--------------------------------)Follow[Apr 14](https://medium.com/@nayeemdev/how-to-send-sms-with-twilio-and-laravel-কিভাবে-twilio-এবং-laravel-এর-মাধ্যমে-sms-পাঠাতে-হয়-5330742cc3fe?source=post_page-----5330742cc3fe--------------------------------) · 2 min read


**১। প্রথমেই যেনে নেওয়া যাক** [**Twilio**](https://www.twilio.com/) **কী?**

[Twilio](https://www.twilio.com/) হল একটি Communication Service প্লাটফর্ম যেটি ক্যালিফোর্নিয়ার সান ফ্রান্সিসকো ভিত্তিক একটি সংস্থা। ফোন কল করার বা রিসিভ করার জন্য, মেসেজ আদানপ্রদান করার জন্য Twilio API প্রদান করে থাকে।

আমরা আজকে জানব কিভাবে এই Twilio Service টি Laravel এর সাথে Integrate করব এবং নির্দিষ্ট নাম্বার বা নাম্বারগুলোতে মেসেজ পাঠাবো। Twilio কে আমরা অন্য কাজেও ব্যবহার করতে পারি যেমন ফোন নাম্বার ভেরিফিকেশন করতে।

**২। কি কি প্রয়োজন?**

*   অবশ্যই একটা Twilio একাউন্ট থাকতে হবে।
*   Twilio ফোন নাম্বার থাকতে হবে _(আপনি একটা ফ্রি নাম্বার পাবেন Trial একাউন্ট এর জন্য )_
*   Twilio Account SID এবং Auth Token.
*   আপনার [Laravel](https://laravel.com/) এপ্লিকেশন টির ভার্সন 5+ হতে হবে।

**৩। তাহলে আর দেরি কেন শুরু করা যাক।**

*   প্রথমেই একটি [Laravel](https://laravel.com/) প্রজেক্ট Create করে নি।

```
composer create project --prefer-dist laravel laravel twilioSmsDemo
```

*   নিচের কমান্ড এর মাধ্যমে Twilio SDK ইন্সটল করি

```
composer require twilio/sdk
```

*   এবার Twilio এর Configaration Settings গুলো .env তে যোগ করি।

```
TWILIO_SID=ACXXXXXXXXXXXXXXXXXXX
TWILIO_TOKEN=XXXXXXXXXXXXXXXXXXXXX
TWILIO_FROM=APXXXXXXXXXXXXXXXXXXX
```

*   আমরা দুইটি Route তৈরি করব যার একটার কাজ হল View File দেখানো এবং অন্যটির কাজ হল মেসেজ Send এর জন্য Controller এর Method এ নিয়ে যাওয়া।

```
Route::view('/sms', 'sms');
Route::post('/sms', 'SmsController@sendSms')->name('sendSMS');
```

*   View file Setup — এখানে আমরা ফোন/মোবাইল নাম্বার এর জন্য একটা Input এবং মেসেজ এর জন্য একটা TextArea Field নিয়েছি।

** sms.blade.php

```
<html>
      <body>
         <form action="{{ route('sendSMS') }}" method="post">
          @csrf
              @if($errors->any())
              <ul>
                 @foreach($errors->all() as $error)
                     <li> {{ $error }} </li>
                 @endforeach
             </ul>
             @endif
             @if( session( 'success' ) )
                  {{ session( 'success' ) }}
             @endif
            <label>Phone numbers (seperate with a comma [,])</label>
            <input type='text' name='numbers' />
            <label>Message</label>
            <textarea name='message'></textarea>
            <button type='submit'>Send!</button>
      </form>
    </body>
</html>
```

*   এখন আমরা নিচের কমান্ড এর মাধ্যমে একটা Controller Create করব।

```
php artisan make:controller SmsController
```

*   এখন এই Controller এ নিচের Code Add করি।

\*\* SmsController

```
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Twilio\Rest\Client;
use Validator;
class SmsController extends Controller
{
   public function sendSms( Request $request ){
       $validator = Validator::make($request->all(), [
           'numbers' => 'required',
           'message' => 'required'
       ]);
   // Your Account SID and Auth Token from twilio.com/console
       $sid    = env('TWILIO_SID');
       $token  = env('TWILIO_TOKEN');
       $from   = env('TWILIO_FROM';
   $client = new Client( $sid, $token);
   if ( $validator->passes() ) {
   $numbers = explode( ',' , $request->input( 'numbers' ) );
   $message = $request->input( 'message' );
   foreach( $numbers as $key => $number )
           {
               $client->messages->create(
                   $number,
                   [
                       'from' => $from,
                       'body' => $message,
                   ]
               );
           }
   return back()->with( 'success', " messages sent!" );
              
       } else {
           return back()->withErrors( $validator );
       }
   }
}
```
