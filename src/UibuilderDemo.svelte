<script lang="ts">
    import { onMount } from "svelte";
    import uibuilder from "node-red-contrib-uibuilder/src/front-end/uibuilderfe.dev";

    // These are "props" - variables that can be used in a parent component when mounting this component & used in the UI
    export let uibsend = null; // will be initialized in onMount
    export let nrMsg = "";
    export let myGreeting =
        "Send me a msg containing msg.greeting to replace this text.";
    // Defined in main.js
    export let anotherProp = "--";

    /** Simple HTML JSON formatter
     * @param {json} json The JSON or JS Object to highlight
     */
    function syntaxHighlight(json) {
        json = JSON.stringify(json, undefined, 4);
        json = json
            .replace(/&/g, "&amp;")
            .replace(/</g, "&lt;")
            .replace(/>/g, "&gt;");
        json = json.replace(
            /("(\\u[a-zA-Z0-9]{4}|\\[^u]|[^\\"])*"(\s*:)?|\b(true|false|null)\b|-?\d+(?:\.\d*)?(?:[eE][+\-]?\d+)?)/g,
            function (match) {
                var cls = "number";
                if (/^"/.test(match)) {
                    if (/:$/.test(match)) {
                        cls = "key";
                    } else {
                        cls = "string";
                    }
                } else if (/true|false/.test(match)) {
                    cls = "boolean";
                } else if (/null/.test(match)) {
                    cls = "null";
                }
                return '<span class="' + cls + '">' + match + "</span>";
            }
        );
        return json;
    } // --- End of syntaxHighlight --- //

    // Only runs when this component is being mounted (e.g. once, when the page is loaded)
    onMount(() => {
        // Start up the uibuilderfe library
        uibuilder.start();

        // A convenient send function that can be wired direct to events - defined as a prop above
        uibsend = uibuilder.eventSend;

        // Listen for new messages from Node-RED/uibuilder
        uibuilder.onChange("msg", function (msg) {
            console.info("msg received from Node-RED server:", msg);
            // Push an HTML highlighted visualisation of the msg to a prop so we can display it
            nrMsg = syntaxHighlight(msg);
            // Update the greeting if present in the msg
            if (msg.greeting) myGreeting = msg.greeting;
        });
    }); // --- End of onMount --- //
</script>

<div>
    <a href="https://nodered.org/" target="_blank">
        <img
            src="node-red-hexagon.svg"
            class="logo node-red"
            alt="Node-RED Logo"
        />
    </a>
    <a href="https://flows.nodered.org/node/node-red-contrib-uibuilder" target="_blank">
        <img
            src="uibuilder-node-blue-125x125.png"
            class="logo"
            alt="Vite Logo"
        />
    </a>
</div>
<h1>+ Node-RED + uibuilder</h1>

<p>
    Easily create data-driven web UI's for Node-RED using <a
        href="https://svelte.dev"
        target="_blank">Svelte</a
    >, the
    <a
        href="https://insights.stackoverflow.com/survey/2021#section-most-loved-dreaded-and-wanted-web-frameworks"
        target="_blank">most loved web framework</a
    >
</p>

<div class="card">
    <button
        on:click={uibsend}
        data-greeting={myGreeting}
        data-something="this is something"
        title="Uses the uibsend fn and sents both static and dynamic data back to Node-RED"
    >
        Click Me
    </button>
</div>

<pre
    id="msg"
    class="syntax-highlight"
    title="Uses @html because nrMsg contains html highlights">{@html nrMsg}</pre>

<p title="A dynamic greeting that can be update using a msg from Node-RED">
    {myGreeting}
</p>
<p title="Some other dynamic property that main.js might update">
    {anotherProp}
</p>

<style>
    /* These styles will be constrained just to this component by Svelte.
	 * Use the dist/app.css file for any definitions you want shared by all components.
     * 
	 * That might be a good place to import uibuilder's uib-styles.css for example, 
     * but app.css is parsed at build time and uib-styles.css isn't available to
     * us until runtime. So if needed you can include uib-styles.css in index.html.
     * 
	 * If you do, then you can use the CSS variables defined there in here as shown.
	 */
     
     h1 {
		/* `color` Assumes you've loaded ./uib-styles.css; if so, affected h1 tags will be blue */
		color: rgb(var(--uib-color-primary));
	}

    pre {
        margin: auto;
        width: fit-content;
        max-width: 720px;
        overflow-x: auto;
        padding-top: 20px;
    }

    .logo {
        height: 6em;
        padding: 1.5em;
        will-change: filter;
    }
    .logo:hover {
        filter: drop-shadow(0 0 2em #646cffaa);
    }
    .logo.node-red:hover {
        filter: drop-shadow(0 0 2em #ff3e00aa);
    }

    /*#region  Colours for Syntax Highlighted pre's; extracted from uib-styles.css*/
    .syntax-highlight {
        color: white;
        background-color: black;
        padding: 5px 10px;
        font-family: Consolas, "ui-monospace", "Lucida Console", monospace;
        white-space: pre;
        text-align: left;
    }

    /* without the :global() selector these styles will be pruned because they are not used at build time */
    :global(.syntax-highlight > .key) {
        color: #ffbf35;
    }
    :global(.syntax-highlight > .string) {
        color: #5dff39;
    }
    :global(.syntax-highlight > .number) {
        color: #70aeff;
    }
    :global(.syntax-highlight > .boolean) {
        color: #b993ff;
    }
    :global(.syntax-highlight > .null) {
        color: #93ffe4;
    }
    :global(.syntax-highlight > .undefined) {
        color: #ff93c9;
    }
    /*#endregion */
</style>
