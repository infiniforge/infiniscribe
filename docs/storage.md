# InfiniScribe storage integration

InfiniScribe opens Android local files and user-granted Storage Access Framework
(SAF) URIs on its own. It uses `libinfiniselect` to preserve authoritative
resource identity, retain Android grants, and resolve resources when a current
content URI is needed.

When installed, Infinifolder supplies the optional storage provider for its
remote, archive, and filesystem-image sources. Infinifolder owns those backends,
credentials, and private data. InfiniScribe does not include Folder's storage
engine or require the Infinifolder app to launch or build. Without Folder, the
local/SAF workflows remain available.

Viewers may stage a document when a parser requires a seekable or complete local
source. Staging and generated outputs remain owned by InfiniScribe, use its
app-private cache and leases, and do not change the authoritative source URI.
Android provider grants, remote access, rendering, and write-back behavior need
device acceptance in addition to compilation and JVM tests.
