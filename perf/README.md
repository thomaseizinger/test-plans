# libp2p performance benchmarking

This project includes the following components:

- `terraform/`: a Terraform scripts to provision infrastructure
- `impl/`: implementations of the [libp2p perf protocol](https://github.com/libp2p/specs/blob/master/perf/perf.md) running on top of e.g. go-libp2p, rust-libp2p or Go's std-library https stack
- `runner/`: a set of scripts building and running the above implementations on the above infrastructure, reporting the results in `benchmark-results.json`

Benchmark results can be visualized with https://observablehq.com/@libp2p-workspace/performance-dashboard.

## Running via GitHub Action

1. Create a pull request with your changes on https://github.com/libp2p/test-plans/.
2. Trigger GitHub Action for branch on https://github.com/libp2p/test-plans/actions/workflows/perf.yml (see _Run workflow_ button).
3. Wait for action run to finish and to push a commit to your branch.
4. Visualize results on https://observablehq.com/@libp2p-workspace/performance-dashboard.

## Running via local machine

### Provision infrastructure

1. Save your public SSH key as the file `./terraform/modules/short_lived/files/perf.pub`; or generate a new key pair with `make ssh-keygen` and add it to your SSH agent with `make ssh-add`.
2. `cd terraform/configs/local`
3. `terraform init`
4. `terraform apply`
5. `CLIENT_IP=$(terraform output -raw client_ip)`
6. `SERVER_IP=$(terraform output -raw server_ip)`

### Build and run implementations

_WARNING_: Running the perf tests might take a while.

1. `cd runner`
2. `npm ci`
3. `npm run start -- --client-public-ip $CLIENT_IP --server-public-ip $SERVER_IP`

### Deprovision infrastructure

1. `cd terraform/configs/local`
2. `terraform destroy`

## Adding a new implementation or a new version

1. Add implementation to `impl/`.
    - For a new implementation, create a folder `impl/<your-implementation-name>/`
    - For a new version of an existing implementation, create a folder `impl/<your-implementation-name>/<your-implementation-version>`.
    - In that folder include a `Makefile` that builds an executable and stores it next to the `Makefile` under the name `perf`.
    - Requirements for the executable:
      - Running as a libp2p-perf server
        - Command line flags
          - `--run-server`
      - Running as a libp2p-perf client
          - Input via command line
            - `--server-ip-address`
            - `--transport` (see `runner/versions.ts` for possible variants)
            - `--upload-bytes` number of bytes to upload per stream.
            - `--download-bytes` number of bytes to download per stream.
          - Output
            - Logging MUST go to stderr.
            - Measurement output is printed to stdout as JSON in the form of:
              ```json
              {"latency": 0.246442851}
              ```
              Note that the measurement includes the time to (1) establish the
              connection, (2) upload the bytes and (3) download the bytes.
2. For a new implementation, in `impl/Makefile` include your implementation in the `all` target.
3. For a new version, reference version in `runner/src/versions.ts`.