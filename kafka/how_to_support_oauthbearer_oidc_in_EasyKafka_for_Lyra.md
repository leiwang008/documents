# Support OAUTH_BEARER/OIDC TOKEN in UE-EasyKafka with Lyra Game on platform Windows x86_64?

We are using [UE-EasyKafka](https://github.com/sha3sha3/UE-EasyKafka) to connect the kafka from our unreal engine plugin. The [UE-EasyKafka](https://github.com/sha3sha3/UE-EasyKafka) depends on the [Modern C++ Kafka API](https://github.com/morganstanley/modern-cpp-kafka) which in turn depends on the [Apache Kafka C/C++ client library librdkafka](https://github.com/confluentinc/librdkafka)

I was trying to connect the kafka with OAUTH_BEARER/OIDC TOKEN, I have modified the code with custom token-refresh-callback to [Support OAUTH_BEARER TOKEN Withe EasyKafka for Lyra](https://github.com/leiwang008/documents/blob/main/kafka/how_to_support_oauthbearer_in_EasyKafka_for_Lyra.md), we can  [Generate azure OIDC token](https://github.com/leiwang008/documents/blob/main/kafka/how_to_generate_azure_oidc_token.md) manually and use the generated token directly. But the problem is that the token will expire and the connection will break in some time.   

I checked the [kafka source code](https://github.com/confluentinc/librdkafka), it already supports the OAUTH_BEARER/OIDC TOKEN by default, we can provide the necessary properties like 'client.id', 'client.secret', 'scope' and 'token.url' etc. and the rdkafka will retrieve the OIDC token from the server automatically. The implementation is at [rdkafka oauthbearer OIDC](https://github.com/confluentinc/librdkafka/blob/6eaf89fb124c421b66b43b195879d458a3a31f86/src/rdkafka.c#L2329) which calls the [rdkafka oauthbearer OIDC callback](https://github.com/confluentinc/librdkafka/blob/6eaf89fb124c421b66b43b195879d458a3a31f86/src/rdkafka_sasl_oauthbearer_oidc.c#L227) to get token automatically.  

But [UE-EasyKafka](https://github.com/sha3sha3/UE-EasyKafka) doesn't support that yet, I have to fully UPGRADE the UE-EasyKafka to use the [Modern C++ Kafka v2024.07.03](https://github.com/morganstanley/modern-cpp-kafka/releases/tag/v2024.07.03) and [Apache librdkafka v2.4.0](https://github.com/confluentinc/librdkafka/releases/tag/v2.4.0)

To correctly update the [UE-EasyKafka](https://github.com/sha3sha3/UE-EasyKafka) we should probably do the following things

1. put all the [modern c++ kafka header files](https://github.com/morganstanley/modern-cpp-kafka/tree/main/include/kafka) into the **ThirdParty/include** folder of project [UE-EasyKafka](https://github.com/sha3sha3/UE-EasyKafka)
2. update the [UE-EasyKafka](https://github.com/sha3sha3/UE-EasyKafka) source code to get consistent with the modern cpp kafka.
3. build the [Apache Kafka C/C++ client library librdkafka](https://github.com/confluentinc/librdkafka) to get the dependencies like librdkafka.lib, librdkafkacpp.lib and put them into the **ThirdParty/lib/Wind64** folder of project [UE-EasyKafka](https://github.com/sha3sha3/UE-EasyKafka)

The first and second step are relatively easy, just copy some files and modify some code to get them consistent. The third step was not easy for me and I also asked for [help from the EasyKafka Founder](https://github.com/sha3sha3/UE-EasyKafka/issues/9), I met many problem and I will write down my experience :-)  

# Build the librdkafka-2.4.0 and use the libraries in Lyra Game on platform Windows x86_64

1. I got the librdkafka2.4.0 source code from https://github.com/confluentinc/librdkafka/releases/tag/v2.4.0
2. Go to the **win32 folder** and open the **librdkafka.sln** with the Visual Studio 2022
3. Then try to build it by "**Build->Build Solution**", this will download a lot of dependencies  
There will be some build errors for some projects (the examples, tests projects etc.)  
`The build tools for v143 (Platform Toolset = 'v143') cannot be found. To build using the v143 build tools, please install v143 build tools.`  
the main c-project **librdkafka, librdkafkacpp** got built successfully and that is what we need, so I just ignore those build errors.
4. By default, the build will be for "**win32**", we need "**X64 build**", go to "**Build->Config Manager ...**", change the "**Active solution platform**" to "**x64**" and change the "**Active solution configuration**" to "**Release**"
5. By default, the build will be "**Dynamic Library (.dll)**", we probably need a **"Static Library" (everything in one big .lib file)**, right-click the "**librdkafak**" and select "**Properties**", and go to "**General -> Configuration Type**" and change it to "**Static library (.lib)**".  
![Generate Static Libraries](img/generated_library_type.png?qc_blockWidth=393&qc_blockHeight=595)  
6. Then right-click the project "**librdkafka**" and select "**ReBuild**", and we will get the release build "**librdkafka.lib**" in folder like "win32\outdir\v143\x64\Release", copy this "librdkafka.lib" to "UE-EasyKafka\Source\ThirdParty\KafkaLib\lib\Win64" and modify "**KafkaLib.Build.cs**" to use this library instead of the old one.
7. Then we do the same thing for project '**librdkafkacpp**' to build and get static library **librdkafkacpp.lib**, and replace the old one in EasyKafka plugin.
8. When I build our plugin code, I got the following errors  
`Severity Code Description Project File Line Suppression State Details Error LNK2001 unresolved external symbol __imp_crc32 LyraStarterGame C:\repository\lyra-demo-kafka\LyraStarterGame\Intermediate\ProjectFiles\librdkafka.lib(rdkafka_admin.obj) 1`  
So I checked the "librdkafka" solution, I found that it depends some installed libraries which have been put in C:\sdk\librdkafka-2.4.0\vcpkg_installed\x64-windows\x64-windows\lib (libcurl.lib, libcrypto.lib, libssl.lib, zlib.lib,zstd.lib etc., **these are dynamic libraries by default, cannot be used directly**) and copied them to the "UE-EasyKafka\Source\ThirdParty\KafkaLib\lib\Win64" and modified the KafkaLib.Build.cs to include them  
9. After all these modifications, I successfully built the whole project
10. But when I try to launch the project, it says that the module "EasyKafka" cannot be loaded  
`Plugin 'EasyKafka' failed to load because module 'EasyKafka' could not be loaded. There may be an operating system error or the module may not be properly set up.`  
11. I noticed that there are some .dll files (libcrypto-3-x64.dll, libssl-3-x64.dll, zlib1.dll, libcurl.dll, zstd.dll, the **.dll file of the dynamic library**) in the folder C:\repository\github\librdkafka\vcpkg_installed\x64-windows\x64-windows\bin, I copied them to the folder "UE-EasyKafka\Source\ThirdParty\KafkaLib\bin\Win64" and then include them as below
	```
		string binPath = Path.Combine(ModuleDirectory, "bin/Win64");
		RuntimeDependencies.Add(Path.Combine(binPath, "libcrypto-3-x64.dll"));
		RuntimeDependencies.Add(Path.Combine(binPath, "libssl-3-x64.dll"));
		RuntimeDependencies.Add(Path.Combine(binPath, "zlib1.dll"));
		RuntimeDependencies.Add(Path.Combine(binPath, "libcurl.dll"));
		RuntimeDependencies.Add(Path.Combine(binPath, "zstd.dll"));
	```  
12. But I still cannot get the EasyKafka plugin loaded. So I tried to use the static libraries (big .lib file) as dependencies, not the dynamic dependencies (small .lib + .dll file). We can run the following command to get the static dependencies libraries, pay attention to **--triplet "x64-windows-static"**, normally it is **--triplet "x64-windows"** (this will get the dependencies as dynamic libraries small .lib + .dll file)  
	```
	vcpkg install  --x-wait-for-lock --triplet "x64-windows-static" --vcpkg-root "C:\repository\github\vcpkg\\" "--x-manifest-root=C:\sdk\librdkafka-2.4.0\\" "--x-install-root=C:\sdk\librdkafka-2.4.0\vcpkg_installed\x64-windows-static\\"
	```  
	we can also do this in the Visual-Studio, right click the **librdkafka** and click the **Properties**, go to the "**Configuration Properties->cvpkg->Use Static Libraries**" and choose **Yes**  
	![Use Static Libraries](img/use_static_libraries.png?qc_blockWidth=393&qc_blockHeight=595)  
	
	Then if we build this **librdkafka** project, we will see the command in the console 
	```
	vcpkg install  --x-wait-for-lock --triplet "x64-windows-static" --vcpkg-root "C:\repository\github\vcpkg\\" "--x-manifest-root=C:\sdk\librdkafka-2.4.0\\" "--x-install-root=C:\sdk\librdkafka-2.4.0\vcpkg_installed\x64-windows-static\\"
	```
	and the static dependencies libraries will downloaded in folder **C:\sdk\librdkafka-2.4.0\vcpkg_installed\x64-windows-static\x64-windows-static\lib**  

13. Then copy all these static dependencies libraries (libcrypto.lib, libcurl.lib, libssl.lib, zlib.lib and zstd.lib) into EasyKafka lib folder **UE-EasyKafka\Source\ThirdParty\KafkaLib\lib\Win64** and modify the **KafkaLib.Build.cs** to include these libraries
	```
	    string LibPath = Path.Combine(ModuleDirectory, "lib/Win64");
        PublicAdditionalLibraries.Add(Path.Combine(LibPath, "libcrypto.lib"));
        PublicAdditionalLibraries.Add(Path.Combine(LibPath, "libcurl.lib"));
        PublicAdditionalLibraries.Add(Path.Combine(LibPath, "libssl.lib"));
        PublicAdditionalLibraries.Add(Path.Combine(LibPath, "zlib.lib"));
        PublicAdditionalLibraries.Add(Path.Combine(LibPath, "zstd.lib"));
	``` 
14. Then if we build the Lyra project, we will still get the following compilation errors
`Severity	Code	Description	Project	File	Line	Suppression State	Details Error	LNK2005	ERR_add_error_data already defined in libcrypto.lib(libcrypto-lib-err.obj)	LyraStarterGame	C:\repository\lyra-demo-kafka\LyraStarterGame\Intermediate\ProjectFiles\libcrypto.lib(err.obj)	1		`  
This is because we also include the Unreal Engine's "**OpenSSL**" module in **KafkaLib.Build.cs** , and there are conflicts between libcrypto.lib/libssl.lib(Version: 3.0.8) and UE5'S OpenSSL(C:\Program Files\Epic Games\UE_5.3\Engine\Binaries\ThirdParty\OpenSSL), just comment out the following code, and the build will pass successfully. 
	```
        PublicDependencyModuleNames.AddRange(
        new string[]
        {
            "OpenSSL",
            "zlib"
        }
        );	
	```  

15. Finally we can run the game with the EasyKafka plugin loaded successfully, PINGO!
16. Next I tried to update the header file librdkafka.h of version v2.4.0 to "UE-EasyKafka\Source\ThirdParty\KafkaLib\include\librdkafka", when I build the project, I got errors like the following
	```
	Severity	Code	Description	Project	File	Line	Suppression State	Details
	Error	LNK2019	unresolved external symbol __imp_rd_kafka_err2str referenced in function "public: virtual class std::basic_string<char,struct std::char_traits<char>,class std::allocator<char> > __cdecl kafka::ErrorCategory::message(int)const " (?message@ErrorCategory@kafka@@UEBA?AV?$basic_string@DU?
	```
	I checked that the function "**rd_kafka_err2str**" does exist in the librdkafka.h, this drove me crazy!  
	Then I copied the content of new librdkafka.h a little bit by bit and carefully compared with the old librdkafka.h, finally I found the reason, the old header file has this **#if 1 // #ifdef LIBRDKAFKA_STATICLIB**, as we are using the static library, I modified so and the errors were gone.
	```
	#if 1 // #ifdef LIBRDKAFKA_STATICLIB
	#define RD_EXPORT
	#else
	```

17. The following a sample code to create a kafka producer with oauthbearer/oidc settings

```
	EasyKafka = GEngine->GetEngineSubsystem<UEasyKafkaSubsystem>()->GetEasyKafka();

	TMap<EKafkaProducerConfig, FString> KafkaConfiguration =
	{
		{EKafkaProducerConfig::MESSAGE_TIMEOUT_MS,5000},
		{EKafkaProducerConfig::REQUEST_TIMEOUT_MS,5000}
	};
	KafkaConfiguration.Add(EKafkaProducerConfig::BOOTSTRAP_SERVERS, "localhost:9092");
	KafkaConfiguration.Add(EKafkaProducerConfig::SASL_MECHANISM, "OAUTHBEARER");
	KafkaConfiguration.Add(EKafkaProducerConfig::SASL_OAUTHBEARER_METHOD, "oidc");
	KafkaConfiguration.Add(EKafkaProducerConfig::SASL_OAUTHBEARER_CLIENT_ID, "<your_client_id>");
	KafkaConfiguration.Add(EKafkaProducerConfig::SASL_OAUTHBEARER_CLIENT_SECRET, "<your_client_secret>");
	KafkaConfiguration.Add(EKafkaProducerConfig::SASL_OAUTHBEARER_SCOPE, "<your_app_scope>");
	KafkaConfiguration.Add(EKafkaProducerConfig::SASL_OAUTHBEARER_TOKEN_ENDPOINT_URL, "<oauthbearer_token_url>");

	KafkaConfiguration.Add(EKafkaProducerConfig::SECURITY_PROTOCOL, "SASL_PLAINTEXT");

	EasyKafka->GetProducer()->CreateProducer("localhost:9092", ""/*user*/, ""/*password*/, KafkaConfiguration, true, (int)EKafkaLogLevel::ERR);
	EasyKafka->GetProducer()->ProduceRecord("your-topic", "your message", 1008 /*RecordId*/, true);
```
