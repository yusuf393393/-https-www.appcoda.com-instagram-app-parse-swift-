version: 2
jobs:
  build:
    working_directory: ~/dkhamsing/open-source-ios-apps
    parallelism: 1
    shell: /bin/bash --login
    environment:
      CIRCLE_ARTIFACTS: /tmp/circleci-artifacts
      CIRCLE_TEST_REPORTS: /tmp/circleci-test-results
    docker:
    - image: circleci/build-image:ubuntu-14.04-XXL-upstart-1189-5614f37
    steps:
    - checkout
    - run: mkdir -p $CIRCLE_ARTIFACTS $CIRCLE_TEST_REPORTS
    - run:
        working_directory: ~/dkhamsing/open-source-ios-apps
        command: rm -f dkhamsing/open-source-ios-apps/.rvmrc; echo 2.4.0 > dkhamsing/open-source-ios-apps/.ruby-version; rvm use 2.4.0 --default
    - restore_cache:
        keys:
        - v1-dep-{{ .Branch }}-
        - v1-dep-master-
        - v1-dep-
    - save_cache:
        key: v1-dep-{{ .Branch }}-{{ epoch }}
        paths:
        - vendor/bundle
        - ~/virtualenvs
        - ~/.m2
        - ~/.ivy2
        - ~/.bundle
        - ~/.go_workspace
        - ~/.gradle
        - ~/.cache/bower
    - run: gem install json_schema
    - run: validate-schema .github/schema.json contents.json
    - run: ruby .github/osia_validate_categories.rb
    - store_test_results:
        path: /tmp/circleci-test-results
    # Save artifacts
    - store_artifacts:
        path: /tmp/circleci-artifacts
    - store_artifacts:
        path: /tmp/circleci-test-results
  deploy:
    working_directory: ~/dkhamsing/open-source-ios-apps
    parallelism: 1
    shell: /bin/bash --login
    docker:
    - image: circleci/build-image:ubuntu-14.04-XXL-upstart-1189-5614f37
    steps:
    - checkout
    - run:
        working_directory: ~/dkhamsing/open-source-ios-apps
        command: rm -f dkhamsing/open-source-ios-apps/.rvmrc; echo 2.4.0 > dkhamsing/open-source-ios-apps/.ruby-version; rvm use 2.4.0 --default
    - restore_cache:
        keys:
        - v1-dep-{{ .Branch }}-
        - v1-dep-master-
        - v1-dep-
    - save_cache:
        key: v1-dep-{{ .Branch }}-{{ epoch }}
        paths:
        - vendor/bundle
        - ~/virtualenvs
        - ~/.m2
        - ~/.ivy2
        - ~/.bundle
        - ~/.go_workspace
        - ~/.gradle
        - ~/.cache/bower
    - run: ruby .github/osia_convert.rb
    - run: ./.github/deploy.sh
#     - run: gem install delete_my_tweets
#     - run: ruby .github/osia_tweet_clean.rb
workflows:
  version: 2
  osia:
    jobs:
      - build
      - deploy:
          requires:
            - build
          filters:
            branches:
              only: master
              
