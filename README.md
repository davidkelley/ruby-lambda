# λ

This PoC aims to implement micro event handlers in Ruby; receiving requests from HTTP and SQS sources, performing operations of varying complexity and then parsing a response.

My idea, is that each handler is implemented inside a separate repository, then packaged as an individual handler and uploaded to a centralised storage mechanism, such as S3. Any external gems that the handler requires are packaged locally and required as such.

Each handler is defined with a unique control path and can include varying meta data for audit and monitoring processes. Below is an example of how a simple handler could be defined:

```
# 1
λ.| 'users/find' do

  # 2
  version "0.0.1"

  # 3
  def lookup(id)
    # perform some database lookup
  end

  # 4
  id = event.query.fetch(:id, nil)

  if !id
    # 5
    context(FAIL, error: "Missing ?id=")
  end

  # 6
  context(OK, lookup(id))

end
```

1. This is how each handler identifies itself within the system. In this case the handler is defined as `users/find`; this identifying path could be used as part of a URL path, a MQTT channel message handler or set up as a handler for SQS.

2. Various meta data can be added to the block that can assist with auditing and monitoring processes, such as implementing a certain amount of version control.

3. Methods can be defined inside the block, just as any other normal Ruby block.

4. The block itself, is executed inside the context of an instance of `λ::Handle`, which is constructed passing options and data through from the received event or request. For example, `event.query<Hash>` might contain HTTP Query Parameters.

5. The `context` method, similar to AWS Lambda allows you to pass through a response from the handler. First indicating success or failure, as well as the body of the response.

## Deployment

The `λ` gem would contain a small CLI, that would enable it to setup a Puma server, as well as an SQS message listener (which could be set to poll on a certain queue). It would run inside a Docker container, linked with another container that would inject a directory via a volume, containing the handler files (as downloaded from S3 or another location).

The Docker container managing the handlers would download files from S3 (or another source), as well as using SupervisorD to sync the files periodically, to ensure that handlers are up-to-date. This method, ensures that `λ` is not dependent upon handlers from a specific location or directory.
