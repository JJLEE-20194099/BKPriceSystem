
[***WATCH: DEMO VIDEO***](https://www.youtube.com/watch?v=n0p-MyRImiQ&t=336s)


***0. System Architecture***
![Data_System](https://hackmd.io/_uploads/HybZ3L1oA.png)


![DATN_Price_Prediction_System](https://hackmd.io/_uploads/ByZWhU1oR.png)


***1. Collect, clean and store data: (CRAWLBOT SERVER)***

+ [Source Code](https://github.com/JJLEE-20194099/datn-prepare-data)
+ How to run the program:
    + You access the terminal to get the source code from github
    + Clone Repo:
    ```json=
    git clone https://github.com/JJLEE-20194099/datn-prepare-data.git
    ```
    + Install the library:
    ```python=
    pip install -r requirements.txt
    ```
    + Set up Kafka Queue: Install kafka service at [link](https://hevodata.com/blog/how-to-install-kafka-on-ubuntu/)
    + Set up broker for kafka service: Set up broker at 3 ports 9092, 9093, 9094
    + Set up Redis
    + Redis port: 6379
    + Redis DB: 5
    + Redis Broker: redis://localhost:6379/5
    + Setup Mongodb and the path to the DB you want to store data
    + You have you can use this Mongo URI public path - [link](mongodb+srv://datn:datn@cluster0.rwxee46.mongodb.net/?retryWrites=true&w=majority&appName=Cluster0)
    + Start the bot:
    ```python=
    !cd into the main folder of the repo
    !sh start_server.sh
    ```
    + Crawlbot server has the endpoint: http://127.0.0.1:8885
    + Run demo:
    ```python=
    python meeyland_get_raw_data.py -> Get real estate information from meeyland
    python meeyland_clean_raw_data.py -> Clean real estate data
    python meeyland_insert_clean_data.py -> Store real estate into database
    ```
![DATN_bkprice_auto_pipeline (1)](https://hackmd.io/_uploads/Skbb38yj0.jpg)
+ Real estate data collection API:
    + Collect data from meeyland site:
        + Get real estate listings from meeyland site:
        ```python=
        curl --location 'http://127.0.0.1:8885/meeyland/crawl_id?page=3'
        ```
        + Get meeyland's key information. The path to get meeyland's real estate information has a random key.
        ```python=
        curl --location 'http://127.0.0.1:8885/meeyland/get_key'
        ```


        + Collect data from mogi page
        ```python=
        curl --location 'http://127.0.0.1:8885/mogi/crawl_url?page=2'
        ```
        + Get mogi's real estate url list by page. You can change the filter to get different pages. After getting the mogi url, the following curl helps you get mogi bds information by url as follows:
        ```json=
        curl --location 'http://127.0.0.1:8885/mogi/crawl_data_by_url?url=https%3A%2F%2Fmogi.vn%2Fquan-binh-tan%2Fmua-nha-hem-ngo%2Fnha-hem-xe-hoi-duong-lien-khu-4-5-4-tang-so-hong-hoan-cong-day-du-id22581832'
        ```

![DATN_MAIN3 (1)](https://hackmd.io/_uploads/SJbZnUyo0.jpg)
    + Crawl data on bds.com
        + Crawl pool URL:
        ```json=
        curl --location 'http://127.0.0.1:8885/batdongsan/crawl_url?page=2'
        ```

        + Crawl content by url:
        ```json=
        curl --location 'http://127.0.0.1:8885/batdongsan/crawl_data_by_url?url=https%3A%2F%2Fbatdongsan.com.vn%2Fban-dat-duong-ngo-chi-quoc-phuong-binh-chieu%2Fcan-ban-gap-lo-thu-duc-58m2-xe-hoi-xay-dung-tu-do-nhinh-2ty-pr39576863'
        ```

    + Collect data from muaban.net
        + Get the list of real estate ids from muaban.net:
        ```json=
        curl --location 'http://127.0.0.1:8885/muaban/crawl_id?offset=6220'. You can replace offset = numbers -> offset like offset in pagination of buying and selling pages
        ```

        + Get real estate information by id:
        ```json=
        curl --location 'http://127.0.0.1:8885/muaban/crawl_data_by_id?id=69337370'
        ```

![DATN_Search_algo.drawio](https://hackmd.io/_uploads/SkZb3UyjA.png)


***2. Manage training dataset, manage feature version to perform data training process in the next stage. (Feature Management Server)***

+ [Source code](https://github.com/JJLEE-20194099/feast-manage-feature-pipeline)

+ How to run the program:
    + You access the terminal where you want to get the source code from git:
    ```json=
    Clone Repo: git clone https://github.com/JJLEE-20194099/feast-manage-feature-pipeline
    ```
    + Install the library:
    ```json=
    pip install -r requirements.txt
    ```
    + Start Feature Management Server:
    ```json=
    cd into the main folder of the repo
    ```
    + Bash command:
        + Start Feast Server: The server performs the management process, creates training data into the feature store for the following training stages:
        ```json=
        !sh start_server.sh
        ```
        + Start Feature Management Server: The server is responsible for performing requests to create feature views, datasets from the outside without directly accessing Feast Server -> Ensure and limit sticky connections between services (you can read the project to better understand the decoupling feature of BKPrice System)
        ```json=
        !sh start_feast.sh
        ```
        + Run Demo:
        ```bash=
        !sh run_exp.sh -> Perform the process of extracting features from real estate data and creating feature views on Feast UI (localhost:5000)
        ```
        ```bash=
        !python create_dataset.py -> Create dataset versions: HCM and HN corresponding to 6 versions of the attribute set mentioned in DATN
        ```

        + API support in Feature Management Server:
            + Create a feature store and proceed with the process of creating feature views to store on Feast:
            ```bash=
            curl --location 'http://127.0.0.1:8885/get_store' \
            --header 'Content-Type: application/json' \
            --data '{
            "path": "feature_repo/"
            }'
            ```
        + Note: path is the path to feature_repo in the main folder of the repo when cloned
        + Get the list of available feature views (Feature view is a dataset that is filtered to retain the attributes in each feature set - there are 6 feature sets corresponding to 6 versions 0 -> 5). View information about each feature set by feature view:

        ```python=
        curl --location 'http://127.0.0.1:8885/get_feature_views'

        curl --location 'http://127.0.0.1:8885/get_feature_names' \
        --header 'Content-Type: application/json' \
        --data '{
        "name": "df4_feature_view"
        }'
        ```
        + Get the list of entities in Feast:
        ```bash=
        curl --location 'http://127.0.0.1:8885/get_entities' \
        --data ''
        ```
        +  In the real estate price prediction problem, the only entity is realestate_id which is the id of each property.
For newly collected data about . Create entity for new data according to realstate_id:
        ```json=
        curl --location 'http://127.0.0.1:8885/register_entity_df' \ --header 'Content-Type: application/json' \ --data '{ "entity_keys": [12, 13], "entity_name": "realestate_id", "frequency": "D", "timestamps": ["1", "2"] }' Create Dataset: curl -- location 'http://127.0.0.1:8885/save_dataset' \ --header 'Content-Type: application/json' \ --data '{ "dataset_name": "dataset_test_api", "dest_folder_path": "./data", "feature_view_names": ["df4_feature_view"]}'
        ```
        + Create dataset from which feature view - Create dataset from which attribute set and store the created dataset in which dest_folder_name. Create dataset from all featureviews, all feature versions and data from 2 cities Hanoi and Ho Chi Minh
        ```bash=
        curl --location --request POST 'http://127.0.0.1:8885/build-offline-batch-data'
        ```

***3. Schedule data crawling, cleaning and storing data. Train the model automatically. (AIRFLOW)***

![airflow1](https://hackmd.io/_uploads/BygW28JsA.png)
![airflow2](https://hackmd.io/_uploads/BkxZ38ks0.png)
![airflow3](https://hackmd.io/_uploads/HkZb2Iks0.png)

+ [Source](https://github.com/JJLEE-20194099/airflow-etl-train-test-pipeline)

+ How to run the program:
    + You access the terminal where you want to get the source code from git

    + Clone Repo:
    ```bash=
    git clone https://github.com/JJLEE-20194099/airflow-etl-train-test-pipeline.git
    ```
    + Install the library:
    ```bash=
    pip install -r requirements.txt
    ```

    + Start the MLFlow server: The MLFlow server is responsible for model management and model training, and performance monitoring. To understand better, you can read DATN
    ```python=
    cd into the main folder of the repo
    !sh start_mlflow.sh
    ```
    ![bar_chart (3) (1)](https://hackmd.io/_uploads/Hkx-38koR.png)
![bar_chart1](https://hackmd.io/_uploads/Sylb2Ukj0.png)
![cronitor_3](https://hackmd.io/_uploads/SkbWn8kiA.png)

    + Start the kafka broker: ensure the data collection and cleaning schedule process, the queue must operate normally
cd into the main folder of the repo
    ```python=
    !sh start_broker.sh (Note that you replace the server.properties config path in the start_broker.sh bash file)
    ```

    + Start airflow:
    ```python=
    cd into the main folder of the repo
    !sh start_airflow.sh
    ```
    + When the above command is finished, the airflow webserver will be run. The airflow scheduler will be run. You can check the status of the airflow scheduler by doing the following:
    ```bash=
    sh sche.sh
    ```

    + Start BKPrice Server: BKPrice server provides APIs for chatbots. Serves the request of chat bot such as training model, collecting data, ...
    ```python=
    !cd to main folder of repo
    !sh start_bkprice.sh
    ```
    + When starting the service -> airflow will automatically schedule to run on the 10th and 19th hour every day to collect data, train real estate data and update the training model. API:
        + Train AI model:
        ```bash=
        curl --location 'http://127.0.0.1:8885/train-ai-model' \
        --header 'Content-Type: application/json' \
        --data '{
        "modelname": "ridge",
        "feature_set_version": 5,
        "city": "hn"
        }'
        ```

        + Real estate valuation:
        ```bash=
        curl --location 'http://127.0.0.1:8885/predict-realestate' \
        --header 'Content-Type: application/json' \
        --data '{ "landSize": 42, "city": "hn", "
        district": "Ha Dong", "ward": "Bien Giang", "street": "National Highway 6", "prefixDistrict": "district", "numberOfFloors": 3, "numberOfBathRooms": 4, "numberOfLivingRooms" : 4, "endWidth": 6, "frontWidth": 6, "frontRoadWidth": 8, "latlon": { "lat": 21.0536611, "lon": 105.8122562 }, "certificateOfLandUseRight": true, "typeOfRealEstate": null , "facade": "twoSideOpen", "houseDirection": "east", "accessibility": "notInTheAlley", "version": "v5" }'
        ```

![demo1](https://hackmd.io/_uploads/S1ZZnUJsA.png)
![gmm_test](https://hackmd.io/_uploads/BkZb28JjA.png)
![line_chart1 (1)](https://hackmd.io/_uploads/Bk--2LkiC.png)
![mlflow_1](https://hackmd.io/_uploads/S1--2UJiC.png)
![mlflow_tn_1](https://hackmd.io/_uploads/rJbWhLJsA.png)
![mlflow3](https://hackmd.io/_uploads/HkZ-38ysR.png)
![mlflow4](https://hackmd.io/_uploads/BJb-nI1sA.png)
![pca](https://hackmd.io/_uploads/SJ-WnLyj0.png)
![price_distribution_v1](https://hackmd.io/_uploads/HyZb3UkjR.png)
![System_overall](https://hackmd.io/_uploads/HylbW281sA.png)


4. ***Chatbot demo: Here I will show you how to setup a simple chatbot to interact with the BKPrice system***


+ [Source](https://github.com/JJLEE-20194099/mlops-function-calling-chatbot)
+ How to run program:
    + You access the terminal where you want to get the source code from git
    + Clone Repo:
    ```bash=
    https://github.com/JJLEE-20194099/mlops-function-calling-chatbot.git
    ```
    + Install the library:
    ```bash=
    pip install - r requirements.txt
    ```
    + Go to notebook file: MLOPS_CHATBOT.ipynb
        + Click Run all
        + Access to http://127.0.0.1:7860: to access the gradient chat interface. You can chat according to the demo sentences in the DATN that can be seen in the links image following:

![image](https://hackmd.io/_uploads/Hy59EPuL0.png)
![image](https://hackmd.io/_uploads/HycTgu_I0.png)
![image](https://hackmd.io/_uploads/ryesOd_IA.png)
![image](https://hackmd.io/_uploads/SyU7cuuL0.png)
![image](https://hackmd.io/_uploads/rkQITKuU0.png)
![image](https://hackmd.io/_uploads/rkMCYj2IA.png)
![image](https://hackmd.io/_uploads/Byoooo3UC.png)
![image](https://hackmd.io/_uploads/ryeyhonLR.png)
![image](https://hackmd.io/_uploads/Hke26ohUA.png)
![image](https://hackmd.io/_uploads/rkVOk228A.png)
![image](https://hackmd.io/_uploads/BJhXen28C.png)
![image](https://hackmd.io/_uploads/rycVG22LR.png )
![image](https://hackmd.io/_uploads/SyPar3nUR.png)
![image](https://hackmd.io/_uploads/S1ePI338R.png)
![image](https://hackmd.io/_uploads/rJaqt32U0.png)
![image](https://hackmd.io/_uploads/SkrVjn3LC.png)