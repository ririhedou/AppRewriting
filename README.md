# App Customization for Security

We provide with rewritten apps, taint-flow analysis results for reproduction.

We rewrite the following apps directly on APK files without source code. We demonstrate the feasibility of our rewriting framework on two well-know benchmarks.

DroisBench: https://github.com/secure-software-engineering/DroidBench

ICC bench: https://github.com/fgwei/ICC-Bench

**We are currently re-constructing the code for rewriting framework**

The tool will be available at:


## ICC Relay to Defend IAC/ICC Vulnerabilities 

IccRelay_DroidBench contains the apps from DroidBench after ICC-relaying based rewriting

IccRelay_ICCBench contains the apps from ICCBench after ICC-relaying based rewriting

The following code snippet (in Jimple IR) shows the IR of the function leakImei after rewriting. 
We mark the inserted function with comments in the code.

The code is located in the method org.arguslab.icc_dynregister1.MainActivity: void leakImei() in the ICC-Bench icc_dynregister1.apk

```Java
private void leakImei()
    {
        org.arguslab.icc_dynregister1.MainActivity $r0;
        android.content.Intent $r1;
        java.lang.Object $r2;
        android.telephony.TelephonyManager $r3;
        java.lang.String $r4;

        $r0 := @this: org.arguslab.icc_dynregister1.MainActivity;

        $r2 = virtualinvoke $r0.<org.arguslab.icc_dynregister1.MainActivity: java.lang.Object getSystemService(java.lang.String)>("phone");

        $r3 = (android.telephony.TelephonyManager) $r2;

        $r4 = virtualinvoke $r3.<android.telephony.TelephonyManager: java.lang.String getDeviceId()>();

        $r1 = new android.content.Intent;

        specialinvoke $r1.<android.content.Intent: void <init>(java.lang.String)>("com.fgwei");

        virtualinvoke $r1.<android.content.Intent: android.content.Intent putExtra(java.lang.String,java.lang.String)>("id", $r4);

         /**
        *We instrument the intent here, by automatical invoking a static method and modify the intent in-place 
        *It supports redirect an activity intent, an service intent and a broadcast intent
        **/
        staticinvoke <iccInstrument.ICCLogAndReplace: void IntentRedirectBroadcast(android.content.Intent)>($r1);

        virtualinvoke $r0.<org.arguslab.icc_dynregister1.MainActivity: void sendBroadcast(android.content.Intent)>($r1);

        return;
    }
    
```

### Proxy App for Security Checking

The two screenshots show the prototype of a proxy app. The proxy app can receive the intent, and inspect the intent (by invoking Get_The_Intent from the above button). Furthermore, the proxy can also reinvoke the intent (by invoking Relay_And_Reinvoke from the blow button). 

In our evaluation, we use the proxy app to successfully prevent ICC vulnerabilities between two apps that communicate with intents. Specifically, the Imei (imei = telephonyManager.getDeviceId(); return null if tested in an emulator) information  is detected in the intent.

The right screen shows an implicit intent is re-constructed and re-invoked. The candidate receiver components are consistent with components from the original app. 

<img src="https://github.com/ririhedou/AppRewriting/blob/master/Taintflow/screen1.png" height="500" width="300"><img src="https://github.com/ririhedou/AppRewriting/blob/master/Taintflow/screen.png" height="500" width="300">


## Logging for Dynamic Monitoring


log_DroidBench contains the apps from DroidBench after log-based rewriting

log_ICCBench contains the apps from ICCBench after log-based rewriting

Logging base rewriting is more straight forward and high feasible. 
Logging based rewriting does not affect the control and data dependencies.

The following log information that is captured by ADB logcat shows how we could use logging-based rewriting to inspect intents and other sensitive sinks. 

The action and data inside an intent (the app is in the ICC-Bench) are captured by our logging-based rewriting. 

```bash
02-24 14:12:50.289  3444  3444 E [rewrite]Log intent: Intent { act=test_action2 cat=[test_category3,test_category4] dat=amandroid://fgwei:8888/xxx/xxx.com typ=test/type (has extras) }
02-24 14:12:50.289  3444  3444 E [rewrite]Log intent: Bundle[{data=null}]
02-24 14:12:50.289  3444  3444 E [rewrite]Log intent: Action = test_action2
02-24 14:12:50.289  3444  3444 E [rewrite]Log intent: Type = test/type
02-24 14:12:50.289  3444  3444 E [rewrite]Log intent: Data = amandroid://fgwei:8888/xxx/xxx.com
02-24 14:12:50.289  3444  3444 E [rewrite]Log intent: DataString = amandroid://fgwei:8888/xxx/xxx.com
```
It is worth to note that the logging based rewriting is easily extended to support dynamic checking. 

By implementing a sensitivity checking function for the logged data, our logging based rewriting can terminate the
sink invocation at runtime. 

## Security Usage of Risk Metrics with Rewriting

/Taintflow/Demo_Bench includes a benchmark app Button1.apk for demonstrating the usage of our risk metrics for rewriting.
We choose this app because the Button1.apk includes a sensitive tain flow from from getDeviceId -> sendTextMessage().
The self permission and aggregate permission are different in the sink sendTextMessage.

This demo shows that our approach can help a security analyst to analyze this sink more easily by providing its flow information and risk value. The risk metrics of sinks significantly reduce the manual efforts to inspect each sink separately. 

In contrast, current Android dynamic permission cannot provide such information. 

Thus, our approach complements with current dynamic permission with more in-depth insights on app security evaluation. 

Furthermore, our rewriting can terminate this sink at runtime. A security analyst can stop the invocation of this sink if the sink is very sensitive (e.g., sending messages to premium number). 

<img src="https://github.com/ririhedou/AppRewriting/blob/master/Taintflow/screen2.png" height="500" width="300">

## Disclaim

This tool is only a research prototype for app rewriting for security.

## Conclusion

In this repository, we present supporting materials for demonstrating the security implications of our approach.

Still, more engineering effort is required, and future research is needed in this direction.


