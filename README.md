# tika-server

Tika server for deployment on AWS Elastic Beanstalk

## Usage

The [Docker Hub image](https://cloud.docker.com/repository/registry-1.docker.io/joeltg/tika-server) should auto-build on every repo push.

Deploying to AWS is just uploading `Dockerrun.aws.json` to Elastic Beanstalk. 😎

- `IPFS_URL` is a [multiaddr](https://github.com/multiformats/multiaddr) like `"/dns4/my-ipfs-node.com/tcp/5001"`
- `DATABASE_URL` is a fully-qualified postgres URI
- `AWS_REGION` is e.g. `"us-east-1"`
- `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` are self-explanatory
- `CONFIGURATION_ID` is the _name of the S3 notification handler_ that is generating the events

## Configuration

- `/Dockerrun.aws.json`
  - `-spawnChild` is documented [here](https://wiki.apache.org/tika/TikaJAXRS#Making_Tika_Server_Robust_to_OOMs.2C_Infinite_Loops_and_Memory_Leaks). "This starts tika-server in a child process, and if there's an OOM, a timeout or other catastrophic problem with the child process, the parent process will kill and/or restart the child process."
  - `-JXmx1g` sets the max heap for the spawned child process at 1GB.
  - `-JXms256m` sets the initial heap for the spawned child process at 256MB.

## Assertions

In `static/` there are two JSON-LD documents `tika-reference.json` (aka `dweb:/ipfs/QmYyRieED9hv4cVH3aQcxTC6xegDZ9kXK2zLxqHAjtBvc7`) and `tika-provenance.json`. These contain "background" knowledge about Tika that are referenced in the provenance of the assertions we generate.

Specifically, we attribute the resulting transcript and metadata documents to `dweb:/ipfs/QmYyRieED9hv4cVH3aQcxTC6xegDZ9kXK2zLxqHAjtBvc7#_:c14n29` - the [`prov:SoftwareAgent`](https://www.w3.org/TR/prov-o/#SoftwareAgent) that is the Tika software application - with the [`prov:qualifiedAssociation`](https://www.w3.org/TR/prov-o/#qualifiedAssociation) that the software agent had a [`prov:Role`](https://www.w3.org/TR/prov-o/#Role) of `dweb:/ipfs/QmYyRieED9hv4cVH3aQcxTC6xegDZ9kXK2zLxqHAjtBvc7#_:c14n61` (for metadata) or `dweb:/ipfs/QmYyRieED9hv4cVH3aQcxTC6xegDZ9kXK2zLxqHAjtBvc7#_:c14n21` (for text extraction). These "roles" correspond to REST API endpoints that are structured as [schema.org EntryPoints](https://schema.org/EntryPoint) and derived from the [HTML API docs](https://gateway.underlay.store/ipfs/QmQofqmV8FHDpaEVVEwtnBv78pVirdswmcSD2oVZzeSokL) that the Tika server serves from `GET "/"` by default. These are frighteningly & admittedly unwieldy: in the future you'll be able to paste these URIs into the Underlay Playground to get explorable visualizations (both from the source document and from subsequent published references). These sorts of references are a low-level representation that should rarely be seen; it's our job to build better tools for referencing them.

`dweb:/ipfs/QmYyRieED9hv4cVH3aQcxTC6xegDZ9kXK2zLxqHAjtBvc7` (aka `tika-reference.json`) is pinned to [the cluster](https://gateway.underlay.store/ipfs/QmYyRieED9hv4cVH3aQcxTC6xegDZ9kXK2zLxqHAjtBvc7) and should be considered **stable, to be changed only when absolutely necessary**. `tika-provenance.json` contains provenance _about_ `tika-reference.json` (via explicit reference to `dweb:/ipfs/QmYyRieED9hv4cVH3aQcxTC6xegDZ9kXK2zLxqHAjtBvc7` as a digital document), citing the HTML API reference (that Tika itself generates!) as its source. _In the (near) future we should [sign](https://web-payments.org/vocabs/security#LinkedDataSignature2015) (with some public KFG key) this document and publish it as well,_ but it's not necessary to get the Prior Art Archive working (unlike `tika-reference.json`, whose hash we need to use in our assertions).

`tika-context.json` is copied from and documented at [this Gist](https://gist.github.com/joeltg/f066945ee780bfee769a26cea753f255).
