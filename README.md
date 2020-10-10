# KillRoboCallers
This project is simply a document describing how I managed to eliminate robocallers.  I welcome pull requests for better or more complete descriptions.

#  How I killed Robocallers with AWS, Ooma, and Google Voice
Situation:  I've had the same home phone number for over ten years, and have been getting blasted by robocallers.  I use NoMoRobo, and still 3-5 get through daily.

I also have an iPhone with a phone number I've had for a few years that was being hit a few times a week.

I also have an old Google Voice number I use on rare occasions that gets hit with a dozen robocalls every week.

## Things I tried
I tried these things before landing on the solution described before:
- NoMoRobo (paid): Still lets a lot of robocallers through
- "Silence Unknown Callers" in iOS 13+: No robocallers, but legitimate callers were also being blocked and sent to voicemail
- "Do Not Disturb" in Google Voice:  I could still get texts, but all callers to Google Voice went to voicemail

## The solution
Prerequisites:  
- The ability to create a new and unknown phone number for your phone
- The ability to forward all calls to the legacy phone number to a new place
- Basic familiarity with AWS

### Big picture
1. Callers call your legacy phone number
2. The call is forwarded directly to an AWS Connect phone number
3. AWS Connect requests the caller press a key to proceed
4. The caller presses a key and the call is forwarded to you new and unknown phone number
5. Your phone rings

### Configuration
#### AWS
Log in to AWS and create an AWS Connect account.  You can skip most of the wizard, as we'll be manually configuring things.

Go into Routing -> Contact Flows and create a new, blank, contact flow.

Create a simple flow with `Get Customer Input` and `Transfer to Phone Number`.  

The customer input should announce `Press 2 to continue` with a single option to press 2, and the Transfer to Phone Number should disable `Contact Flow After Disconnect`. 

Note you can customize this further to provide more advanced phone tree options, or a directory of your family's cell phone numbers, optionally setting the caller ID to the endpoint phone number to bypass "Silence Unknown Callers" in iOS 13+.

Next, go to Routing -> Phone Numbers and claim a new phone number.  Under `Contact Flow / IVR` select the Contact Flow you created above.

Note this appears to be free under AWS's free usage tier today, but that may change in the future.

#### Ooma
Note: This section can apply to different phone providers, but will obviously need adjusting.

1. Log in to Ooma Telo
2. Go to Preferences -> Phone Numbers and select `Add Phone Number`
3. Complete the form to add your new additional phone number
4. Go to the `Call Forwarding` page
5. Under your main number: Un-check `Ring Telo` and `Ring Mobile App` if they are checked and check the `Ring ...` option, selecting the great, and inputting your AWS Connect endpoint phone number from the AWS section
7. Under your new, unknown number: Check `Ring Telo` and whatever other options you want

#### Next Steps
You're set - now anybody calling your legacy phone number will be routed to AWS to confirm they are capable of pressing a number before being allowed to ring your home phone number.

I then integrated my legacy Google Voice phone number by setting it up to forward to my legacy home phone number, so it gets the same workflow.  I set Google Voice to allow it to answer immediately.

I then took it a step further and set up more menus to allow people to press 2 to ring our home phone or 3 to ring our cell phones, at which point they can press 1 for me, 2 for my wife, etc.  For calling me, I set the caller ID to the AWS Connect endpoint phone number, added that to my contacts and re-enabled `Silence Unknown Callers`. 

At some point in the future, I'd like to integrate a lambda function to whitelist known good callers (my wife or mother, for instance) so they ring directly through without the need to press numbers to continue.
