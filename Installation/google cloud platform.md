# Google Cloud Platform

## 使用Container-optimized Google Compute Engine images快速开始。

1. 进入[Google Cloud Console][1] 创建一个新的Cloud Project with
   [Compute Engine enabled][2]

2. 下载和配置 [Google Cloud SDK][3] 来使用你的项目，通过如下命令:

        $ curl -sSL https://sdk.cloud.google.com | bash
        $ gcloud auth login
        $ gcloud config set project <google-cloud-project-id>

3. 使用最新的启动新的实例 [Container-optimized image][4]:
   (select a zone close to you and the desired instance size)

        $ gcloud compute instances create docker-playground \
          --image container-vm \
          --zone us-central1-a \
          --machine-type f1-micro

4. 使用SSH链接到实例:

        $ gcloud compute ssh --zone us-central1-a docker-playground
        docker-playground:~$ sudo docker run hello-world
	Hello from Docker.
	显示在你的安装界面证明其正常工作。
	...

阅读更多请点击[deploying Containers on Google Cloud Platform][5].

[1]: https://cloud.google.com/console
[2]: https://developers.google.com/compute/docs/signup
[3]: https://developers.google.com/cloud/sdk
[4]: https://developers.google.com/compute/docs/containers#container-optimized_google_compute_engine_images
[5]: https://developers.google.com/compute/docs/containers
