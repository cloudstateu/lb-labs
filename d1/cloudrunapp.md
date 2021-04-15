<img src="../../../img/logo.png" alt="Chmurowisko logo" width="200"  align="right">
<br><br>
<br><br>
<br><br>

# Cloud Run lab

## LAB Overview

#### In this lab you will create Cloud Run and devops process for it.

## Task 1: Create New Application

1. Open GCP console through https://console.cloud.google.com/.
2. Open Cloud Shell
3. Execute command: ```dotnet new web -o helloworld-csharp```
4. Change directory to helloworld-csharp.

5. Update the CreateHostBuilder definition in Program.cs to listen on the port defined by the PORT environment variable:

```
using System;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.Hosting;

namespace helloworld_csharp
{
    public class Program
    {
        public static void Main(string[] args)
        {
            CreateHostBuilder(args).Build().Run();
        }

        public static IHostBuilder CreateHostBuilder(string[] args)
        {
            string port = Environment.GetEnvironmentVariable("PORT") ?? "8080";
            string url = String.Concat("http://0.0.0.0:", port);

            return Host.CreateDefaultBuilder(args)
                .ConfigureWebHostDefaults(webBuilder =>
                {
                    webBuilder.UseStartup<Startup>().UseUrls(url);
                });
        }
    }
}
```
6. Create a file named Startup.cs and paste the following code into it:
```
using System;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.AspNetCore.Http;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;

namespace helloworld_csharp
{
    public class Startup
    {
        // This method gets called by the runtime. Use this method to add services to the container.
        // For more information on how to configure your application, visit https://go.microsoft.com/fwlink/?LinkID=398940
        public void ConfigureServices(IServiceCollection services)
        {
        }

        // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
        public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
        {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }

            app.UseRouting();

            app.UseEndpoints(endpoints =>
            {
                endpoints.MapGet("/", async context =>
                {
                    var target = Environment.GetEnvironmentVariable("TARGET") ?? "World";
                    await context.Response.WriteAsync($"Hello {target}!\n");
                });
            });
        }
    }
}
```
7. Your app is finished and ready to be containerized and uploaded to Container Registry.

## Task 2: Contenerized App
1. To containerize the sample app, create a new file named Dockerfile in the same directory as the source files, and copy the following content:
```
# Use Microsoft's official build .NET image.
# https://hub.docker.com/_/microsoft-dotnet-core-sdk/
FROM mcr.microsoft.com/dotnet/core/sdk:3.1-alpine AS build
WORKDIR /app

# Install production dependencies.
# Copy csproj and restore as distinct layers.
COPY *.csproj ./
RUN dotnet restore

# Copy local code to the container image.
COPY . ./
WORKDIR /app

# Build a release artifact.
RUN dotnet publish -c Release -o out


# Use Microsoft's official runtime .NET image.
# https://hub.docker.com/_/microsoft-dotnet-core-aspnet/
FROM mcr.microsoft.com/dotnet/core/aspnet:3.1-alpine AS runtime
WORKDIR /app
COPY --from=build /app/out ./

# Run the web service on container startup.
ENTRYPOINT ["dotnet", "helloworld-csharp.dll"]
```
3. To exclude files produced via local dotnet build operations from upload to Cloud Build add a .gcloudignore file in the same directory as the sample app's source files:

```
# The .gcloudignore file excludes file from upload to Cloud Build.
# If this file is deleted, gcloud will default to .gitignore.
#
# https://cloud.google.com/cloud-build/docs/speeding-up-builds#gcloudignore
# https://cloud.google.com/sdk/gcloud/reference/topic/gcloudignore

**/obj/
**/bin/

# Exclude git history and configuration.
.git/
.gitignore
```
4. Build your container image using Cloud Build, by running the following command from the directory containing the Dockerfile:
```
gcloud builds submit --tag gcr.io/PROJECT-ID/helloworld
```
where PROJECT-ID is your GCP project ID. You can get it by running ```gcloud config get-value project```.

Upon success, you will see a SUCCESS message containing the image name (gcr.io/PROJECT-ID/helloworld). The image is stored in Container Registry and can be re-used if desired.

## Task 3: Deploying to Cloud Run

1. Deploy using the following command:
```
gcloud run deploy --image gcr.io/PROJECT-ID/helloworld --platform managed
```
3. Visit your deployed container by opening the service URL in a web browser.


## Task 3. Cleanup resources.

1. Delete previously created instance from the list by marking it and using trash icon from the menu.

## END LAB


