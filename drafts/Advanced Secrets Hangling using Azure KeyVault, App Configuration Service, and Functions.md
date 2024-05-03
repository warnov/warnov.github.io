# Advanced Secrets Handling using Azure KeyVault, App Configuration Service, and Functions

1. `kv-isolated-tests` created ok
2. `app-conf-isolated-tests` created ok
3. `FxIsolatedTest`created ok
4. `FxIsolatedTest` referencing app-conf-isolated-tests through string connection plain in the `fx app`configuration ok (through a simple app config variable stored in the App Configuration Service)

   ```c#
   namespace SimpleFx.Fx
   {
       public class FxIsolatedTest(ILogger<FxIsolatedTest> logger, IConfiguration configuration)
       {
           private readonly ILogger<FxIsolatedTest> _logger = logger;
           private readonly IConfiguration _configuration = configuration;
   
   
           [Function("FxIsolatedTest")]
           public IActionResult Run([HttpTrigger(AuthorizationLevel.Function, "get", "post")] HttpRequest req)
           {
               _logger.LogInformation("C# HTTP trigger function processed a request.");
               var message = _configuration["confkey"];
               return new OkObjectResult($"Welcome to Azure Functions! {message}");
           }
       }
   }
   ```

   


5. `FxIsolatedTest` bringing a value from app-conf-isolated-tests ok

## Referencing KeyVault for direct access from Azure Function Configuration

Let's try now to get the connection string for the `App Configuration Service` from the `KeyVault`so no one with access to the `App Configuration Service`can access it directly.

First, let's do a simple test getting any secret from the `KeyVault` as if there are errors during the `host` initialization, the `function `won't run and it would be very hard to diagnose the error. This first test will allow us to confirm that the `function `has direct access to the `KeyVault` and after this, we would try to reference a secret directly from the ``host initialization`.

- [x] Add a secret to the `kv`: ok: `secret1`

- [x] Reference the secret from the `function`: ok

- [x] Inject the `key vault` service in the `host` initialization. Result **ok** with this code:
  ```C#
  var host = new HostBuilder()
      .ConfigureFunctionsWebApplication()
      .ConfigureAppConfiguration((hostingContext, config) =>
      {
          var settings = config.Build();
          config.AddAzureAppConfiguration(options =>
          {
              options.Connect(settings["ConnectionStrings:AppConfig"]);
          });
  
          var keyVaultEndpoint = new Uri("https://kv-isolated-tests.vault.azure.net/");
          config.AddAzureKeyVault(keyVaultEndpoint, new DefaultAzureCredential());
      })
      .ConfigureServices(services =>
      {
          services.AddApplicationInsightsTelemetryWorkerService();
          services.ConfigureFunctionsApplicationInsights();
      })
      .Build();
  
  host.Run();
  ```

  In this first attempt, just a direct connection with the `kv` is made. This is not intended to handle secrets through the `App Configuration Service`. This is just for getting its connection string. The endpoint of the `kv `is required in this case. This value, it is safe to put it then in the `Azure Function` configuration.

- [x] Test Locally as the dev machine already has access to the `kv` **ok**

- [x] Give permissions in Azure to the `function`over the `kv`

  - [x] Assign an identity to the `Fx`
  - [x] Give permissions to this identity in keyvault

- [x] Publish the function and test it: **Success** Just giving secrets user permissions to the function identity as a managed identity directly from the portal without any use of the cli

  

## Using KeyVault to bring the App Configuration Service connection string for the Azure Function

- [x] Add the secret `connection string` to the `kv`: ok: `isolated-app-conf-connstring`

- [x] Modify `Startup.cs` to read the `connection string` of the `app configuration service` from the `kv`: Observe that here, the secret is read directly from the `kv` it is not necessary to even have the setting in the `Function Confir Settings` (this is required for blob and queue triggered functions to read the base connection string from the `kv`)

- [x] Remove local environment variables of this `connection string` or change the code to call another name (in this way we will be sure that the `connection string` will be coming from the `kv`)

  ```C#
  var host = new HostBuilder()
      .ConfigureFunctionsWebApplication()
      .ConfigureAppConfiguration((hostingContext, config) =>
      {
          var keyVaultEndpoint = new Uri("https://kv-isolated-tests.vault.azure.net/");
  
          // Load the connection string for Azure App Configuration from Azure Key Vault
          config.AddAzureKeyVault(keyVaultEndpoint, new DefaultAzureCredential());
  
          var settings = config.Build();
  
          // Use the connection string to connect to Azure App Configuration
          config.AddAzureAppConfiguration(options =>
          {
              options.Connect(settings["isolated-app-conf-connstring"])
                     .ConfigureKeyVault(kv =>
                     {
                         kv.SetCredential(new DefaultAzureCredential());
                     });
          });
      })
      .ConfigureServices(services =>
      {
          services.AddApplicationInsightsTelemetryWorkerService();
          services.ConfigureFunctionsApplicationInsights();
      })
      .Build();
  
  host.Run();
  ```

  

- [x] Test Locally

- [x] Remove Azure Function variables of this `connection string`

- [x] Publish new version

- [x] Test on Azure

## Access secret from KeyVault through App Configuration Service

- [x] Add new secret to `kv` to then reference it from `AppConf`:  `secret2`
- [x] Reference the previous in the `AppConf`   `kv-referenced-secret`
- [x] Update code to query this setting   `kv-referenced-secret`
- [x] Test locally
- [x] Publish
- [x] Test on Azure

## Conclusion

The error I was having very probably was because I haven't gave the `function `a direct mechanism to connect to KV to get the connection string for the `App Configuration Service`. Let's try to correct that in the application.

## Correction in the current solution

After making some corrections, the function still not working so here are the next steps:

### FxCustomerSpaceCreator

- [x] Delete current `kv: `

  - [x] `app-conf-customer-spaces` must be recreated here
  - [x] `customerspacestagging-connection-string` must be recreated and named `customer-spaces-stagging`
  - [x] old `customer spaces connection strings` will be also deleted
  - [x] re-creation inside correct `rg`

- [x] Delete current `app-conf-customer-spaces `

  - [x] `key-vault-url`
    - [x] should be moved to function configuration
  - [x] `location`
    - [x] `eastus2`
  - [x] `resource-group`
    - [x] `RG_CUSTOMER_SPACES`
    - [x] re-creation as `rg_customer_spaces`
  - [ ] `sa-customer-spaces-stagging` (better to take it from the `kv`)
  - [x] `subs-id`
    - [x] `ff735c14-ae5a-4322-bbcf-4155bb380106`

- [x] Delete current `IntelappExternalServicesCustomerSpaces` it will now be `Intelapp.CustomerSpaces`

  - [x] Create `FxVer `to test all connections
  - [x] Re-create `FxCustomerSpaceCreator`
    - [x] Test it locally
    - [x] Publish it
    - [x] Test it on Azure and watch it failing because lack of permissions to modify elements in the RG
    - [x] Assign Permissions
      - [x] **IMPORTANT**: Refine permissions of the function so it can only create storage accounts inside the **RG**: Tested with the custom role `Custom Storage Account Creator` and it seemed to work. 
      - [x] Let's try tomorrow after full propagation 
        - It looks that propagation can take many minutes
        - List Keys permission was assigned to custom role to try to make this work
        - After many minutes the fix seems to have worked
        - - [ ] Let's try again in a few hours
    - [x] Test Again

  ### FxTargetExpander: Blob Triggered

  #### Exploring basic functionality

  **WARNING:** Executing some proposed configurations here could stop the entire Function App to correctly start. Be careful with each change you deploy.

  This would require an [App Setting referencing the `KV`](https://learn.microsoft.com/en-us/azure/app-service/app-service-key-vault-references?tabs=azure-cli) to be available as an environment variable so it can be hooked in the creation of the function. Then the reference could point straight to the `keyvault `as `sa-customer-space-staging-cs` is already there. Some tests must be held before actually creating the `Fx`.

  - [x] Add the secret reference
    `@Microsoft.KeyVault(VaultName=myvault;SecretName=mysecret)`
    `@Microsoft.KeyVault(VaultName=kv-customer-spaces;SecretName=sa-customer-space-staging-cs)`

  - [x] Modify `FxVer `to test locally that the reference can be called

  - [x] Republish: Check nothing got wrong

  - [x] Test new functionality on Azure

  - [x] Proceed to create the blob triggered function

  - [x] Test it locally

  - [x] Deploy function

  - [x] Test it on Azure
  
  
  
  #### Complementing the full functionality
  
  - [x] Let's bring all the code generated before for the `target expander`.
  - [x] Main functionality completed and working right on Azure
  - [x] Put a message in the `profile-processing` queue in the `saintelappopenai `storage account for Nova to process it
  
  #### Conclusion
  
  The `AppSetting `reference to `KV `works easier than expected. Just operating like described above, allows us to connect the `Fx `to the storage account with the connection string referenced in the KV so no one can seen that connection string. Also, it was discovered that ***<u>you can see the logs of the functions</u>*** without `app insights`, going into the *Code+Test* functionality in the *Developer* section of the particular function inside the *function app* in the *Azure Portal*. There you can connect to the *Filesystem Logs* that are the ones that present the information from the `ILogger`.
  
  
  
  
  
  ## FxTargetAnalyzer
  
  - [x] Create queue triggered function
  
  - [x] Use `saintelappopenai ` setting:  and `profile-processing`queue
  
  - [x] Publish a simple version and confirm messages from the queue are being processed
  
  - [ ] Add real functionality and observe the behavior of multiple calls against OpenAI when processing a load of messages in the queue
  
    - [x] Add the new `App Configuration Service` reference (the one with `OpenAi Configuration`). It is just needed to put this code in the current host initialization in `Program.cs:`
      ```C#
        // Add a second Azure App Configuration
        config.AddAzureAppConfiguration(options =>
        {
            options.Connect(settings["app-conf-open-ai-cs"])
                   .ConfigureKeyVault(kv =>
                   {
                       kv.SetCredential(new DefaultAzureCredential());
                   });
        });
      ```
  
    - [x] Add the connection string to the `KV`
  
    - [x] Test the deserializing of the message into `a Target Identification` and the access to the `OpenAi Config` locally
  
    - [x] Test this on Azure: Specially the access to the `OpenAI Config`
      At the beginning it failed and the Function App didn't started. The error was that the published function hadn't the access to the `KeyVault` that is referenced for the `OpenAi App Config Service (vaultcert)`. After giving these permissions the function started and worked great!
  
    - [x] Complete the whole functionality and test it. Check the effects of massive parallel calls to `openai`.
  
      
  
    ### ToDos
  
    - [ ] Reduce temperature in hobbies creation
    - [ ] Correct the genre based on the name when generating personality
    - [ ] Add birthday to identifier.json
