---
layout: page
title: Red Hat Kogito
subtitle: Sub-Atomic ER-Demo Process Service using Red Hat Kogito
---
# 1. Overview

# 2. Identified Risks

- Producing and consuming messages from the existing AMQ Streams topics in ER-Demo
- Persisting process state to the existing DataGrid 8 cluster in ER-Demo
- Potentially needing to also implement the "outbox pattern" in  the process service as has been implemented in ER-Demo:   https://www.erdemo.io/process_service_outbox/
- Timers:   is kogito going with quartz or something else for timers ?   ER-Demo makes use of a timer in its business process
- REST workItemHandler:  you mentioned that there is work being done to leverage the REST client component of camel:   awesome.

# 3. Development
# 4. Lessons Learned
# 5. Areas of Improvement
