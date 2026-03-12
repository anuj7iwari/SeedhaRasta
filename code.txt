(function () {

console.log("GraphQL observer running...");

let captured = [];

/* ---------------- FETCH HOOK ---------------- */

const originalFetch = window.fetch;

window.fetch = async function (...args) {

    const response = await originalFetch.apply(this, args);

    try {

        const url = args[0];

        if (typeof url === "string" && url.includes("/api/graphql")) {

            const clone = response.clone();

            clone.text().then(t => {

                try {
                    const json = JSON.parse(t);

                    captured.push({
                        url: url,
                        time: new Date().toISOString(),
                        data: json
                    });

                    console.log("Captured (fetch)", json);

                } catch {}
            });
        }

    } catch {}

    return response;
};




const origOpen = XMLHttpRequest.prototype.open;
const origSend = XMLHttpRequest.prototype.send;

XMLHttpRequest.prototype.open = function(method, url) {

    this._url = url;
    return origOpen.apply(this, arguments);
};

XMLHttpRequest.prototype.send = function(body) {

    this.addEventListener("load", function() {

        try {

            if (this._url && this._url.includes("/api/graphql")) {

                const text = this.responseText;

                try {

                    const json = JSON.parse(text);

                    captured.push({
                        url: this._url,
                        time: new Date().toISOString(),
                        data: json
                    });

                    console.log("Captured (xhr)", json);

                } catch {}

            }

        } catch {}

    });

    return origSend.apply(this, arguments);
};



window.downloadChats = function(trigger){

    if(trigger !== true){
        console.log("Run: downloadChats(true)");
        return;
    }

    const blob = new Blob(
        [JSON.stringify(captured, null, 2)],
        {type:"application/json"}
    );

    const url = URL.createObjectURL(blob);

    const a = document.createElement("a");
    a.href = url;
    a.download = "Main.json";
    a.click();

    URL.revokeObjectURL(url);

    console.log("Dld", captured.length, "responses");
}

})();
