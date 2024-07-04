# Support OAUTH_BEARER TOKEN in UE-EasyKafka with Lyra Game?

We are using [UE-EasyKafka](https://github.com/sha3sha3/UE-EasyKafka) to connect the kafka from our unreal engine plugin. The [UE-EasyKafka](https://github.com/sha3sha3/UE-EasyKafka) depends on the [Modern C++ Kafka API](https://github.com/morganstanley/modern-cpp-kafka) which in turn depends on the [Apache Kafka C/C++ client library librdkafka](https://github.com/confluentinc/librdkafka)

I was trying to implement the code to connect the kafka with OAUTH\_BEARER TOKEN, but [UE-EasyKafka](https://github.com/sha3sha3/UE-EasyKafka) doesn't support that yet, the OAUTH\_BEARER TOKEN was recently added to [Apache Kafka C/C++ client library librdkafka](https://github.com/confluentinc/librdkafka) and [Modern C++ Kafka API](https://github.com/morganstanley/modern-cpp-kafka), BUT [UE-EasyKafka](https://github.com/sha3sha3/UE-EasyKafka) doesn't keep up with it.

To correctly update the [UE-EasyKafka](https://github.com/sha3sha3/UE-EasyKafka) we should probably do the following things

1. build the [Apache Kafka C/C++ client library librdkafka](https://github.com/confluentinc/librdkafka) to get the dependencies like rdkafka.lib, rdkafka++.lib and lz4.lib etc.
2. put all the [morden c++ kafka header files](https://github.com/morganstanley/modern-cpp-kafka/tree/main/include/kafka) and the dependencies (built in the first step) into the ThirdParty folder of project [UE-EasyKafka](https://github.com/sha3sha3/UE-EasyKafka)
3. update the [UE-EasyKafka](https://github.com/sha3sha3/UE-EasyKafka) source code.

I didn't want to upgrade the [UE-EasyKafka](https://github.com/sha3sha3/UE-EasyKafka) completely, so I just started from the second step and copied the necessary files something like below

```txt
        UE-EasyKafka/Source/ThirdParty/KafkaLib/include/kafka/ClientCommon.h
        UE-EasyKafka/Source/ThirdParty/KafkaLib/include/kafka/ClientConfig.h
        UE-EasyKafka/Source/ThirdParty/KafkaLib/include/kafka/Interceptors.h
        modified:   UE-EasyKafka/Source/KafkaConsumer/Private/KafkaConsumer.cpp
        modified:   UE-EasyKafka/Source/KafkaProducer/Private/KafkaProducer.cpp
        modified:   UE-EasyKafka/Source/ThirdParty/KafkaLib/include/kafka/AdminClient.h
        modified:   UE-EasyKafka/Source/ThirdParty/KafkaLib/include/kafka/KafkaClient.h
        modified:   UE-EasyKafka/Source/ThirdParty/KafkaLib/include/kafka/KafkaConsumer.h
        modified:   UE-EasyKafka/Source/ThirdParty/KafkaLib/include/kafka/KafkaProducer.h
        modified:   UE-EasyKafka/Source/ThirdParty/KafkaLib/include/kafka/Properties.h
        modified:   UE-EasyKafka/Source/ThirdParty/KafkaLib/include/kafka/RdKafkaHelper.h
```

The modified files looks like below:

![Modified Source Files](img/modified_files.png?qc_blockWidth=393&qc_blockHeight=595)

and I made some necessary changes to the source codes, everything went smoothly. 

The method **KafkaClient::configInterceptorOnNew()** calls [Apache Kafka C/C++ client library librdkafka](https://github.com/confluentinc/librdkafka)'s method  **rd\_kafka\_interceptor\_add\_on\_broker\_state\_change**, and it is not in the current rdkafka dependencies in my project, so I just commented it out. If I really want it, I have to finish the first step (build the [Apache Kafka C/C++ client library librdkafka](https://github.com/confluentinc/librdkafka) to get the dependencies like rdkafka.lib, rdkafka++.lib and lz4.lib etc.) mentioned above.

- I thought that everything is OK, but when I rebuilt the project, dalala, surprise, I got a build error somthing as below

## **'dynamic\_cast' used on polymorphic type 'kafka::Properties::ValueType::Object' with /GR-; unpredictable behavior may result**

the compiler complained the following code using **dynamic\_cast**

```cpp
            template<class T>
            T& getValue() const { return (dynamic_cast<ObjWrap<T>&>(*object)).value; }
```

There is a funny thing, the unreal engine's code also uses the **dynamic\_cast** in file C:\Program Files\Epic Games\UE\_5.3\Engine\Source\Runtime\CoreUObject\Public\Templates\*\*Casts.h\*\*, it never reports any problem before. But all of sudden it reports the same problem as above after upgrading the **morden c++ kafka code**, don't know why.

```cpp
		if constexpr (!TIsCastable<FromValueType>::Value || !TIsCastable<ToValueType>::Value)
		{
			// This may fail when dynamic_casting rvalue references due to patchy compiler support
			return dynamic_cast<To>(Arg);
		}
```

# To fix this problem, we need to `enable Run-Time Type Information (RTTI)` to **`/GR`**, but how to add this option  `/GR` ? NO NO, the following 2 ways will not work for unreal engine project.

**I tried 2 ways**

1. Modify the project config properties as below, these Build, Rebuild, Compile Command Line will finally call the unreal build script.

![add build option /GR](img/add_option_to_build_script.png?qc_blockWidth=766&qc_blockHeight=335)

2. Modify the build batch script to add that option, my build script is C:\Program Files\Epic Games\UE\_5.3\Engine\Build\BatchFiles\Build.bat, if you modify the build script, then you don't need to modify the project config properties in the the first way.

```dos
rem the option /GR to enable RTTI dynamic_cast
echo Running UnrealBuildTool: dotnet %UBTPath% %* /GR
dotnet %UBTPath% %* /GR
```

**NO, THIS DIDN'T WORK**. It seems the build got passed, BUT in the log we got message like

```dos
28>Couldn't find target rules file for target '/GR' in rules assembly 'UE5Rules, Version=0.0.0.0, Culture=neutral, PublicKeyToken=null'.
```

The build command is actually as below, I tried to add the option '**/GR**' in different places in this command line, but none worked.

```dos
dotnet "C:\Program Files\Epic Games\UE_5.3\Engine\Binaries\DotNET\UnrealBuildTool\UnrealBuildTool.dll" LyraEditor Win64 Development -Project="C:\github\LyraStarterGame\LyraStarterGame.uproject" -WaitMutex -FromMsBuild -Rebuild
```

# **FIX: To enable Run-Time Type Information (RTTI) in an Unreal Engine project:**

specifically for a project like`LyraEditor`, you need to modify the build configuration settings. **This is usually done in the** **`LyraEditor.Build.cs`** **file associated with your project, rather than using Visual Studio directly. Unreal Engine uses its own build system, UnrealBuildTool (UBT), to manage compiler settings.**

We already have the following code in EasyKafka.Build.cs, we still need to add these codes to other modules like **KafkaAdmin, KafkaConsumer, KafkaProducer and KafkaLib**, after adding that, the compilation error was gone, finally BINGO!

```csharp
		bEnableExceptions = true;

		if( Target.Platform == UnrealTargetPlatform.Win64)
			bUseRTTI = true;//Avoid using RTTI on limux
```

OK, we got the project built successfully with the upgraded code of morden-kafka. Next we need to modify the Easy-Kafka to support oauth-bearer-token, let's take the producer as an example (we should probably modify the same for the consumer):

1. Define the call-back function '**onOauthbearerTokenRefresh**'  (required by morden-kafka) in **KafkaProducer/Public/KafkaProducer.h**

```cpp
// oauthbearerConfig will be a json string as below, you can use your own format
// {"Token":"your.token.value", "PrincipalName":"alice", "LeftTimeMS":9999999999999, "extensions": {"a":"val", "b":"val"}}
static SaslOauthbearerToken onOauthbearerTokenRefresh(const std::string& oauthbearerConfig);
// need the following function to convert the "extensions" to a map<string, string>
static std::map<std::string, std::string> ConvertJsonObjectToStdMap(TSharedPtr<FJsonObject> JsonObject);
```

2. Implement these 2 methods in **KafkaProducer/Private/KafkaProducer.cpp**

```cpp
SaslOauthbearerToken FKafkaProducerModule::onOauthbearerTokenRefresh(const std::string& oauthbearerConfig) {
	SaslOauthbearerToken sasltoken;

	// Create a JSON reader
	TSharedRef<TJsonReader<TCHAR>> JsonReader = TJsonReaderFactory<TCHAR>::Create(UTF8_TO_TCHAR(oauthbearerConfig.c_str()));
	// Declare a JSON object to hold the parsed data
	TSharedPtr<FJsonObject> JsonObject;
	// Parse the JSON string
	if (FJsonSerializer::Deserialize(JsonReader, JsonObject) && JsonObject.IsValid())
	{
		// Accessing data from the parsed JSON
		FString token;
		if (JsonObject->TryGetStringField("Token", token)) {
			sasltoken.value = std::string(TCHAR_TO_UTF8(*token));
		}
		else {
			std::string errmsg = "FKafkaProducerModule::onOauthbearerTokenRefresh() Failed to parse the JSON field 'Token'.\n" + oauthbearerConfig;
			UE_LOG(LogKafkaProducer, Error, TEXT("%s"), UTF8_TO_TCHAR(errmsg.c_str()));
			throw std::runtime_error(errmsg);
		}

		FString pricipalName;
		if (JsonObject->TryGetStringField("PrincipalName", pricipalName)) {
			sasltoken.mdPrincipalName = std::string(TCHAR_TO_UTF8(*pricipalName));
		}
		else {
			std::string errmsg = "FKafkaProducerModule::onOauthbearerTokenRefresh() Failed to parse the JSON field 'PrincipalName'.\n" + oauthbearerConfig;
			UE_LOG(LogKafkaProducer, Error, TEXT("%s"), UTF8_TO_TCHAR(errmsg.c_str()));
			throw std::runtime_error(errmsg);
		}

		int64 leftTimeMS;
		if (JsonObject->TryGetNumberField("LeftTimeMS", leftTimeMS)) {
			sasltoken.mdLifetime = std::chrono::milliseconds(leftTimeMS);
		}
		else {
			std::string errmsg = "FKafkaProducerModule::onOauthbearerTokenRefresh() Failed to parse the JSON field 'LeftTimeMS'.\n" + oauthbearerConfig;
			UE_LOG(LogKafkaProducer, Error, TEXT("%s"), UTF8_TO_TCHAR(errmsg.c_str()));
			throw std::runtime_error(errmsg);
		}

		//TSharedPtr<FJsonObject> extensions;
		//extensions = JsonObject->GetObjectField("extensions");
		//sasltoken.extensions = ConvertJsonObjectToStdMap(extensions);
		const TSharedPtr<FJsonObject>* extensions;
		if (JsonObject->TryGetObjectField("extensions", extensions)) {
			sasltoken.extensions = ConvertJsonObjectToStdMap(*extensions);
		}
		else {
			std::string errmsg = "FKafkaProducerModule::onOauthbearerTokenRefresh() Failed to parse the JSON field 'extensions'.\n" + oauthbearerConfig;
			UE_LOG(LogKafkaProducer, Error, TEXT("%s"), UTF8_TO_TCHAR(errmsg.c_str()));
			throw std::runtime_error(errmsg);
		}

		UE_LOG(LogKafkaProducer, Display, TEXT("SaslOauthbearerToken created."));
	}
	else
	{
		std::string errmsg = "FKafkaProducerModule::onOauthbearerTokenRefresh() Failed to parse the JSON string.\n" + oauthbearerConfig;
		UE_LOG(LogKafkaProducer, Error, TEXT("%s"), UTF8_TO_TCHAR(errmsg.c_str()));
		throw std::runtime_error(errmsg);
	}

	return sasltoken;
}

std::map<std::string, std::string> FKafkaProducerModule::ConvertJsonObjectToStdMap(TSharedPtr<FJsonObject> JsonObject)
{
	std::map<std::string, std::string> resultMap;

	if (JsonObject.IsValid())
	{
		for (auto& pair : JsonObject->Values)
		{
			FString key = pair.Key;
			FString value;

			if (pair.Value->Type == EJson::String)
			{
				value = pair.Value->AsString();
			}
			else
			{
				TSharedRef<TJsonWriter<>> writer = TJsonWriterFactory<>::Create(&value);
				// TODO Serialize() needs a key to store this value, when Deserialize, we probably need to get the real value without the key
				FJsonSerializer::Serialize(pair.Value.ToSharedRef(), pair.Key, writer);
			}

			resultMap[std::string(TCHAR_TO_UTF8(*key))] = std::string(TCHAR_TO_UTF8(*value));
		}
	}

	return resultMap;
}
```

3. Then modify the constructor FKafkaProducerModule::CreateProducer to add the oauthbearer callback function

```cpp
	// check for SASL_MECHANISM, if it is OAUTHBEARER, we need to add the bearer-token-callback-function
	// as the value of field 'oauthbearer_token_refresh_cb' into the KafkaProducerProps
	if (KafkaProducerProps->getProperty(kafka::clients::producer::Config::SASL_MECHANISM) == "OAUTHBEARER") {
		KafkaProducerProps->put(kafka::clients::Config::OAUTHBEARER_TOKEN_REFRESH_CB, OauthbearerTokenRefreshCallback(onOauthbearerTokenRefresh));
	}
```

4. Finally we need to create the KafkaProducer with the oauthbearertoekn config string set to property '**sasl.oauthbearer.config**', that is very important, the [librdkafka's function rd\_kafka\_conf\_set\_oauthbearer\_token\_refresh\_cb](https://github.com/confluentinc/librdkafka/blob/bb2843b5ce32c52954fced9b746205c9340bf96f/src/rdkafka.h#L2260) will get its value and assign to the callback function parameter 'const char \***oauthbearer\_config**'.

```cpp
	EasyKafka = GEngine->GetEngineSubsystem<UEasyKafkaSubsystem>()->GetEasyKafka();

	TMap<EKafkaProducerConfig, FString> KafkaConfiguration =
	{
		{EKafkaProducerConfig::MESSAGE_TIMEOUT_MS,5000},
		{EKafkaProducerConfig::REQUEST_TIMEOUT_MS,5000}
	};
	KafkaConfiguration.Add(EKafkaProducerConfig::BOOTSTRAP_SERVERS, "localhost:9092");
	KafkaConfiguration.Add(EKafkaProducerConfig::SASL_MECHANISM, "OAUTHBEARER");
	// set 'sasl.oauthbearer.config', the librdkafka will read it.
	KafkaConfiguration.Add(EKafkaProducerConfig::SASL_OAUTHBEARER_CONFIG, "{ \"Token\":\"your.token.value\", \"PrincipalName\" : \"alice\", \"LeftTimeMS\" : 9999999999999, \"extensions\" : {\"a\":\"val\", \"b\" : \"val\"} }");
	KafkaConfiguration.Add(EKafkaProducerConfig::SECURITY_PROTOCOL, "SASL_PLAINTEXT");


	EasyKafka->GetProducer()->CreateProducer("localhost:9092", ""/*user*/, ""/*password*/, KafkaConfiguration, true, (int)EKafkaLogLevel::ERR);
	EasyKafka->GetProducer()->ProduceRecord("your-topic", "your message", 1008 /*RecordId*/, true);
```

Now we are happy to run the code to connect to the kafka with oauth bearer token. Of cause we need a running kafka server accepting the oauth bearer token for authentication, to run such a kafka server, please refer to my article [How to run kafka in OAUTHBEARER test mode](https://github.com/leiwang008/documents/blob/main/kafka/how_to_run_kafka_in_sasl_plaintext_oauthbearer_with_default_unsecure_token.md)