# TurboRepo NextJs with Github Actions and Deployment on Digital Ocean

This is a showcase repo with following implementations
1. Configure Monorepo
2. Configure Docker build Images separately for each repo[in the monorepo]
3. Set up docker-compose for production
4. Set up Github actions/workflow with following 
    1. Push images on docker hub
    2. Push docker-compose on remote server
    3. Pull and Start the images from the compose