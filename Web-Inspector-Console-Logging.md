Console messages can be sent to the Web Inspector Console from a variety of places:

* from JavaScript [`console`](https://webkit.org/web-inspector/console-object-api/) methods
    * always `MessageSource::ConsoleAPI`
* in the WebProcess by calling `ScriptExecutionContext::addConsoleMessage`
    * unless an `Inspector::ConsoleMessage` is explicitly provided, `MessageType::Log` is used
* in the UIProcess/GPUProcess by sending `Messages::WebPage::AddConsoleMessage`
    * always `MessageType::Log`
* in the NetworkProcess by sending `Messages::NetworkProcessConnection::BroadcastConsoleMessage`
    * always `MessageType::Log`

Console messages can be configured in a few ways:

* `MessageSource` indicates *who*/*what* is dispatching the console message (e.g. `JS` comes from the JavaScript language itself, `ConsoleAPI` comes from [`console`](https://webkit.org/web-inspector/console-object-api/) methods, `CSS` comes from CSSOM APIs or other CSS activity, etc.)
    * Although it may be tempting to just use `JS` for console messages created as a result of calling some JavaScript-exposed API, please use a more specific `MessageSource` as `JS` is really meant for messages related to the JavaScript language itself.  Feel free to create a new `MessageSource` (see below) if no existing one fits.
* `MessageType` indicates the *function* of the console message (e.g. `Log` will log a message, `Clear` will clear the Web Inspector Console, etc.)
    * You should use `Log` unless there is special functionality/UI associated with your console message (e.g. starting performance profiling, clearing the console, etc.).  Each `MessageType` essentially corresponds to a similarly named [`console`](https://webkit.org/web-inspector/console-object-api/) function.
* `MessageLevel` indicates the *importance/severity* of the message (e.g. `Log` is a basic log, `Warning` is for warnings, etc.)
* `const String& message` is the string that is shown in the log (supports [formatting/styling](https://webkit.org/web-inspector/console-object-api/#string-formatting-and-styling))
* `unsigned long requestIdentifier` associates this console message with a particular network request in Web Inspector (if non-zero)
* `const String& url` is the URL of the associated resource that dispatched this console message
* `unsigned line` is the line number in the associated resource that dispatched this console message
* `unsigned column` is the column number in the associated resource that dispatched this console message

Adding new sources/types/levels is possible by modifying all of the following:

* Web Inspector Backend (and the rest of WebKit)
    * *[Source/JavaScriptCore/runtime/ConsoleTypes.h](https://github.com/WebKit/WebKit/blob/main/Source/JavaScriptCore/runtime/ConsoleTypes.h)*
        * `JSC::MessageSource`
        * `JSC::MessageType`
        * `JSC::MessageLevel`
    * *[Source/JavaScriptCore/runtime/ConsoleClient.cpp](https://github.com/WebKit/WebKit/blob/main/Source/JavaScriptCore/runtime/ConsoleClient.cpp)*
        * `static void appendMessagePrefix(StringBuilder&, JSC::MessageSource, JSC::MessageType, JSC::MessageLevel)`
    * [Source/WebKitLegacy/mac/WebCoreSupport/WebChromeClient.mm](https://github.com/WebKit/WebKit/blob/main/Source/WebKitLegacy/mac/WebCoreSupport/WebChromeClient.mm)
        * `static NSString *stringForMessageSource(JSC::MessageSource)`
        * `static NSString *stringForMessageLevel(JSC::MessageLevel)`
* Web Inspector Protocol
    * *[Source/JavaScriptCore/inspector/protocol/Console.json](https://github.com/WebKit/WebKit/blob/main/Source/JavaScriptCore/inspector/protocol/Console.json)*
        * `ChannelSource
            `
        * `ConsoleMessage`
            * `level`
            * `type`
    * *[Source/JavaScriptCore/inspector/ConsoleMessage.cpp](https://github.com/WebKit/WebKit/blob/main/Source/JavaScriptCore/inspector/ConsoleMessage.cpp)*
        * `static Inspector::Protocol::Console::ChannelSource messageSourceValue(JSC::MessageSource)`
        * `static Inspector::Protocol::Console::ConsoleMessage::Type messageTypeValue(JSC::MessageType)`
        * `static Inspector::Protocol::Console::ConsoleMessage::Level messageLevelValue(JSC::MessageLevel)`
* Web Inspector Frontend
    * *[Source/WebInspectorUI/UserInterface/Models/ConsoleMessage.js](https://github.com/WebKit/WebKit/blob/main/Source/WebInspectorUI/UserInterface/Models/ConsoleMessage.js)*
        * `WI.ConsoleMessage.MessageSource`
        * `WI.ConsoleMessage.MessageType`
        * `WI.ConsoleMessage.MessageLevel`
    * *[Source/WebInspectorUI/UserInterface/Models/IssueMessage.js](https://github.com/WebKit/WebKit/blob/main/Source/WebInspectorUI/UserInterface/Models/IssueMessage.js)*
        * `WI.IssueMessage` (`constructor`)
    * *[Source/WebInspectorUI/UserInterface/Views/ConsoleMessageView.js](https://github.com/WebKit/WebKit/blob/main/Source/WebInspectorUI/UserInterface/Views/ConsoleMessageView.js)*
        * (controls how console messages are rendered)

Past Examples:

* [222692@main](https://commits.webkit.org/222692@main) Web Inspector: provide a way to log messages from the network process
* [210079@main](https://commits.webkit.org/210079@main) Web Inspector: provide a way to capture a screenshot of a node from within the page

## Automatic Logging from `WTFLogChannel`

* Web Inspector Backend (and the rest of WebKit)
    * *[Source/WebCore/inspector/agents/page/PageConsoleAgent.cpp](https://github.com/WebKit/WebKit/blob/main/Source/WebCore/inspector/agents/page/PageConsoleAgent.cpp)*
        * `void PageConsoleAgent::getLoggingChannels(ErrorString&, RefPtr<JSON::ArrayOf<Inspector::Protocol::Console::ChannelSource>>&)`
    * *[Source/WebCore/dom/Document.cpp](https://github.com/WebKit/WebKit/blob/main/Source/WebCore/dom/Document.cpp)*
        * `static MessageSource messageSourceForWTFLogChannel(const WTFLogChannel& channel)`
* Web Inspector Protocol
    * *[Source/JavaScriptCore/inspector/protocol/Console.json](https://github.com/WebKit/WebKit/blob/main/Source/JavaScriptCore/inspector/protocol/Console.json)*
        * `ChannelSource`
* Web Inspector Frontend
    * *[Source/WebInspectorUI/UserInterface/Controllers/ConsoleManager.js](https://github.com/WebKit/WebKit/blob/main/Source/WebInspectorUI/UserInterface/Controllers/ConsoleManager.js)*
        * `WI.ConsoleManager.prototype.initializeLogChannels`
    * *[Source/WebInspectorUI/UserInterface/Models/LoggingChannel.js](https://github.com/WebKit/WebKit/blob/main/Source/WebInspectorUI/UserInterface/Models/LoggingChannel.js)*
        * `WI.LoggingChannel` (i.e. the `constructor`)
    * *[Source/WebInspectorUI/UserInterface/Views/LogContentView.js](https://github.com/WebKit/WebKit/blob/main/Source/WebInspectorUI/UserInterface/Views/LogContentView.js)*
        * `WI.LogContentView` (i.e. `constructor`)
        * `WI.LogContentView.prototype._scopeFromMessageSource`
        * `WI.LogContentView.Scopes`
    * *[Source/WebInspectorUI/UserInterface/Views/SettingsTabContentView.js](https://github.com/WebKit/WebKit/blob/main/Source/WebInspectorUI/UserInterface/Views/SettingsTabContentView.js)*
        * `WI.SettingsTabContentView.prototype._createConsoleSettingsView`

Past Examples:

* [209160@main](https://commits.webkit.org/209160@main) Add MSE logging configuration
* [194925@main](https://commits.webkit.org/194925@main) Web Inspector: Enable WebKit logging configuration and display
