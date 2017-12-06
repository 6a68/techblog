blog post - shot creation flow RAIL instrumentation

——

what are the goals?
- we want screenshots to be an outstanding experience.
- part of that is a great feature set, and part of that is performance.

how do we define our performance goals? use industry standards.
- RAIL: a acronym / mnemonic for some standard HCI measurements.
- neilsen article from 90s, etc.
- these numbers have been around in comp sci / HCI since the dawn of computing.

how do we apply these goals?
- website: pretty straightforward
    - GA gives us DOMContentLoaded and page loaded times
    - since we need the image present on the page to do work, page load is really the key number
- addon: a little harder, we’ll focus on this for now
    - response times: 100ms from interaction to visual update
    - addon code runs in three separate contexts
        - chrome, background page, content page
    - the shot creation flow touches all three contexts
        - walk through / diagram of shot creation flow
    - note, we do already have communication across contexts.
        - we also have metrics code fired from chrome and content to background, where it’s forwarded to our server, which proxies pings before forwarding them to GA. this protects our users’ privacy.
    - first, naive solution: pass around events with start times between contexts
        - but: this is kinda clunky and sprinkles calculation code all over the place
    - nicer solution: use existing metrics code, add internal-only events & add a filter to track start and end events, then fire those separately as user timings
        - since we already have metrics code that centrally handles all events in one spot,
        - piggyback off this existing stream of events:
        - pass all events through a filter function
            - the filter looks for start events, when it sees them, marks the time
            - filter then looks for either an end event, or a cancel event
                - end event: one step in the shot creation flow is complete, subtract current time from start time, then send the elapsed time to the GA User Timing endpoint
                - cancel event: the user aborted shot creation, so clear out the start time and start over
            - one point to note: a few end points didn’t have existing events
                - example: iframe show/hide. no need to send that to GA, we don’t care about counting the number of iframe show/hide events.
                - solution: create “internal” events, routed through the GA event stream, but not actually sent to GA (filter them out)
- show actual data from Data Studio?
    - initial click to showing overlay is super slow
        - we have some lazy-loading code that we might be able to remove
        - failing that, we could also immediately load an overlay, then replace it with the "Real” overlay when that’s ready
            - perceived performance ftw
    - time to download big images is super slow


