# ADR-002 Distribute JS via CDN

| Property        | Value                                 |
| --              | --                                    |
| *Components*    | Backend, Web                          |
| *Related*       |                                       |
| *Superceded by* |                                       | 

## Context

*What is the issue that we're seeing that is motivating this decision or change?*

Currently the javascript components used in the library registry and circulation manager are served to clients by passing them through 
the Flask app. I will use examples from the circulation manager in this document, but the library registry contains roughly the same code. 

- *circulation/api/admin/routes.py*
  ```python
  @app.route('/admin/static/circulation-web.js')
  @returns_problem_detail
  def admin_js():
      directory = os.path.join(os.path.abspath(os.path.dirname(__file__)), "node_modules", "simplified-circulation-web", "dist")
      return app.manager.static_files.static_file(directory, "circulation-web.js")

  @app.route('/admin/static/circulation-web.css')
  @returns_problem_detail
  def admin_css():
      directory = os.path.join(os.path.abspath(os.path.dirname(__file__)), "node_modules", "simplified-circulation-web", "dist")
      return app.manager.static_files.static_file(directory, "circulation-web.css")
  ```
- *circulation/api/controller.py*
  ```python
  class StaticFileController(CirculationManagerController):
      def static_file(self, directory, filename):
          cache_timeout = ConfigurationSetting.sitewide(
              self._db, Configuration.STATIC_FILE_CACHE_TIME
          ).int_value
          return flask.send_from_directory(directory, filename, cache_timeout=cache_timeout)
  ```

The code for the javascript and css file are built in the docker container by pulling a package in from NPM based on the 
`package.json` file located in the `api/admin` folder. 

Serving the files this way poses several challenges:

1. NPM isn't really meant to distribute the final app, its more meant to pull libraries into an app. The final webapp is assumed to be 
   deployed outside of NPM. 
2. It requires nodejs to be installed into the docker container to build app, even though it isn't needed or used for actually serving 
   the files. This bloats the docker container and provides a larger attack surface. 
3. Flask isn't ideally suited to the goal of serving static files.
4. Having `package.json` and `package-lock.json` included in the Python repostories, makes github issue security alerts for updates in 
   in these repositories, even though the updates should really be applied in the upstream JS repositories.

## Alternatives Considered

*Any alternatives considered any details about why they were set aside.*

- Pulling the files into the container and serving them using uWSGI / Flask. This is currently how we are serving these files.
  It was set aside for the reasons outlined in the section above.
- Flask supports a configuration option to send files using the `X-Sendfile` header. This isn't directly supported by nginx, 
  but it looks like we could write a configuration that would support it. We would still need to go into python to serve the 
  files, but at least they would be served through nginx. This would help to mitigate #3, but still leaves the other issues 
  unaddressed.
- Using [s3-spa-upload](https://www.npmjs.com/package/s3-spa-upload) NPM package to upload the files to a s3 bucket and 
  serve them directly from there. This option solves most of the issues we have, but requires additional infrastructure 
  to host the files in S3.

## Decision

*What is the change that we're proposing and/or doing? This may have several subsections giving more detail about the decision.*

We will continue to package releases and push them to NPM using github CI. However instead of pulling these releases into the 
docker container using NPM, we will serve the files out of a CDN that mirrors the files in NPM.

We will use the [jsdelivr](https://www.jsdelivr.com/) CDN since it mirrors every package that is published to NPM. 

In the HTML templates included in the Python packages we will reference the package urls out of jsdelivr. The URLs will 
look like: 

- https://cdn.jsdelivr.net/npm/@thepalaceproject/circulation-admin@0.0.2/dist/circulation-admin.css
- https://cdn.jsdelivr.net/npm/@thepalaceproject/circulation-admin@0.0.2/dist/circulation-admin.js

These published build artifacts can then directly be pulled into the backend service by URL. The version of the package 
we are using can be bumped by changing which version is pulled into the HTML template in a particular service. 

## Consequences

*What becomes easier or more difficult to do because of this change?*

This will make the docker builds of the library-registry and circulation backends easier and more secure. It will mean that 
you don't have to install node and npm in order to use package the python backend, or run it in debug mode. 

It will improve performance of our webservers since python threads are not spending time serving javascript files. Pages 
should load faster for clients since they will be able to cache these files basically forever since they have a unique 
path per version.

The developer experience will be impacted minimally, we can preserve the existing behaviour where local files are served when 
running the server in development mode.
