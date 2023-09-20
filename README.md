# TypeChat.NET

TypeChat.NET is an **experimental project** from the [Microsoft Semantic Kernel](https://github.com/microsoft/semantic-kernel) team. TypeChat.NET brings the ideas of [TypeChat](https://github.com/microsoft/TypeChat) to .NET. 

TypeChat.NET provides **cross platform** libraries that help you build natural language interfaces with language models using strong types, type validation and simple type safe programs (plans). Strong typing may help make software that uses language models more deterministic and reliable.    
```
// Translates user intent into strongly typed Calendar Actions
var translator = new JsonTranslator<CalendarActions>(
    new LanguageModel(Config.LoadOpenAI())
);
```

TypeChat.NET is in **active development** with frequent updates. The framework and programming model will evolve as the team explores the space and incorporates feedback. Supported scenarios are shown in the included [Examples](./examples). Documentation will also continue to improve. When in doubt, please look at the code.  

# Assemblies
TypeChat.NET consists of the following assemblies:

* **Microsoft.TypeChat**: Classes that translate user intent into strongly typed and validated objects. 

* **Microsoft.TypeChat.Program**: Classes to synthesize, validate and run  ***JSON programs***. 

* **Microsoft.TypeChat.SemanticKernel**: Integration with Microsoft Semantic Kernel for Language models, Plugins and Embeddings.

* **Microsoft.TypeChat.Dialog**: Classes for working with interactive Agents that have history. 

## Microsoft.TypeChat ##
TypeChat uses language models to translate user intent into JSON that conforms to a schema. This JSON is then validated and deserialized into a typed object. Additional constraint checking is applied as needed.

TypeChat provides:
* Json Translation and validation.
* Schema export: generate schema for .NET Types using **Typescript**, a language designed to express the schema of JSON objects concisely. Schema export supports common scenarios (as shown in the examples) and provides:
  * Runtime export: This is needed for scenarios where the schema may vary for each request, may have to be selected from sub-schemas or include dynamic lists and vocabularies (such as product names, or lists of players in a team).
  * Vocabularies: easy specification of string tables along with support for dynamic loading. 
  
* Classification: lightweight text classifiers used to route requests to child or hierarchical schemas and handlers.
* Extensibility: interfaces let you customize schemas (including hand-written), validators and even language model prompts.

## Microsoft.TypeChat.Program ##
TypeChat.Program translates natural language requests into simple programs (***Plans***), represented as JSON. 

JSON programs can be thought of as a [DSL](https://en.wikipedia.org/wiki/Domain-specific_language) or [Plan](https://learn.microsoft.com/en-us/semantic-kernel/ai-orchestration/planners/?tabs=Csharp), expressed in JSON, with an associated [**grammar**](src/typechat.program/ProgramSchema.ts) that is enforced. JSON programs can be type checked against the APIs they target. They can be then be run using an interpreter, or compiled into .NET code. Both mechanisms enforce type safety.

TypeChat.Program includes:
* Program Translator: translates user intent into programs that follow the [Program Grammar](src/typechat.program/ProgramSchema.ts)
* Program Interpreter: runs programs generated by ProgramTranslator using an interpreter.
* Program Compiler: uses the dynamic language runtime (DLR) to compile programs/plans with type checking. Compilation diagnostics are used to repair programs. 
* Program C# Transpiler/Compiler (experimental): transpiles programs into C# and compile them into assemblies. Compilation diagnostics are used to repair programs.  
```
// Translates user intent into typed Programs that call
// methods on a Math API
_api = new MathAPI();
_translator = new ProgramTranslator<IMathAPI>(
    new LanguageModel(Config.LoadOpenAI()),
    _api
);
 ```

## Microsoft.TypeChat.SemanticKernel ##
TypeChat.SemanticKernel provides out-of-the-box bindings for language models, plugins and embeddings to Typechat.NET and TypeChat.Net examples.

TypeChat.SemanticKernel include classes for:
* **Json Programs for Plugins**: turn registered plugins into **APIs** that Json programs can target See the [Plugins Example](examples/Plugins/Program.cs).
* Language model and embeddings access: all TypeChat examples use the Semantic Kernel to call models and generate embeddings. 
* Embeddings classes: Easy to use in-memory tables with nearest neighbor matching support. These make it easy to build lightweight classifiers and routers. See the [SchemaHierarchy example](examples/SchemaHierarchy/Program.cs) for details.
 
## Microsoft.TypeChat.Dialog
(Early)
TypeChat.Dialog demonstrates how TypeChat.NET may be used for strongly typed interactions with message passing Agents or Bots. These agents can include features such as built in interaction history. 

TypeChat.Dialog includes support for:
* Agents, Agents with History
* Messages, Message Streams
```
// Create an agent with history
_agent = new AgentWithHistory<HealthDataResponse>(new LanguageModel(Config.LoadOpenAI()));
```

# Getting Started 

## Prerequisite: Open AI
* **Open AI Language Models**: TypeChat.NET and its examples currently require familiarity with and access to language models from Open AI. 
* TypeChat.NET has been tested with and supports the following models: 
    * gpt-35-turbo
    * gpt-4
    * ada-002
* Some examples and scenarios will work best with gpt-4
* Since TypeChat.NET uses the Semantic Kernel, models from other providers ***may*** be used for experimentation.

## Building

* Visual Studio 2022. 
  * Load **typechat.sln** from the root directory. 
  * Restore packages
  * Build

* dotnet build
  * Launch a command prompt / terminal
  * Go to the root directory of the project
  * dotnet build Typechat.sln

## Nuget Packages

(Coming)

## Examples

To see TypeChat in action, explore the [TypeChat example projects](./examples). The list below describes which examples will best introduc which concept. Some examples or scenarios may work best with gpt-4.

* Hello World: The [Sentiment](./examples/Sentiment) example is TypeChat's Hello World and a minimal introduction to JsonTranslator. 

* JsonTranslator and Schemas: 
  
  * CoffeeShop: Natural language ordering at a coffee shop
  * Calendar: Transform language into calendar actions
  * Restaurant: Order at a pizza restaurant

* Hierarchical schemas and routing:
  * MultiSchema: dynamically route user intent to other 'sub-apps'
  * SchemaHierarchy: A Json Translator than uses multiple child JsonTranslators

* Json Programs and TypeChat.Program:
  * Math: Turn user requests into calculator programs
  * Plugins (program synthesis that target Semantic Kernel Plugins)

* Typed Agents:
  * Healthdata
  
Each example includes an **input.txt** with sample input. Pass the input file as an argument to run the example in **batch mode**. 

## Api Key and Configuration
To use TypeChat.net or run the examples, you need an **API key** for an Open AI service. Azure Open AI and the Open AI service are both supported.

### Configure Api Key for examples
* Go to the **[examples](./examples)** folder in the solution
* Make a copy of the [appSettings.json](./examples/appSettings.json) file and name it **appSettings.Development.json**. Ensure it is in the same folder as appSettings.json
* appSettings.Development.json is a local development only override of the settings in appSettings.json and is **never** checked in.
* Add your Api Key to **appSettings.Development.json**. 

A typical appSettings.Development.json will look like this:
```
// For Azure Open AI service
{
  "OpenAI": {
    "Azure": true,
    "ApiKey": "YOUR API KEY",
    "Endpoint": "https://YOUR_RESOURCE_NAME.openai.azure.com",
    "Model": "gpt-35-turbo"  // Name of Azure deployment
  }
}

// For Open AI Service:
{
  "OpenAI": {
    "Azure": false,
    "ApiKey": "YOUR API KEY",
    "Endpoint": "https://api.openai.com/v1/chat/completions",
    "Model": "gpt-3.5-turbo"  // Name of Open AI model
  }
}
```

## OpenAIConfig
TypeChat accesses language models using the [LanguageModel](./src/typechat.sk/LanguageModel.cs) class. The OpenAIConfig class provides the configuration it needs.  

You can initialize OpenAIConfig via your application's configuration system (see examples for how) or from environment variables. 

See [OpenAIConfig.cs](./src/typechat.sk/OpenAIConfig.cs) for a list full list of :
  * Configurable properties
  * Supported environment variables.
```
// Your configuration 
OpenAIConfig config = Config.LoadOpenAI();
// Or from config
config = OpenAIConfig.FromEnvironment();

var model = new LanguageModel(config);
```

### Using Semantic Kernel directly
You can also initialize LanguageModel using an IKernel object you created using a KernelBuilder.
```
const string modelName = "gpt-35-turbo";
new LanguageModel(_kernel.GetService<IChatCompletion>(modelName), modelName);
```

# License

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.

# Trademarks

This project may contain trademarks or logos for projects, products, or services. Authorized use of Microsoft 
trademarks or logos is subject to and must follow 
[Microsoft's Trademark & Brand Guidelines](https://www.microsoft.com/en-us/legal/intellectualproperty/trademarks/usage/general).
Use of Microsoft trademarks or logos in modified versions of this project must not cause confusion or imply Microsoft sponsorship.
Any use of third-party trademarks or logos are subject to those third-party's policies.
