This repo contains ansible code to deploy a fully configured jenkins server

When cloning from github, simply run:

    rake

When using galaxy, simply run:

    ansible-galaxy install Azulinho.azulinho-jenkins-fully-baked


To consume this role, either add the following settings to your group_vars all,
or create a 'wrapper_role' that sets these variables in the
role/wrapper_role/vars/main.yaml.


# jenkins configuration settings
jenkins:
  version: 1.592-1.1
  dest: /opt/jenkins
  lib: /var/lib/jenkins
  port: 8080
  prefix: /jenkins
  cli_dest: '/opt/jenkins/jenkins-cli.jar' # Jenkins CLI destination
  updates_dest: '/opt/jenkins/updates_jenkins.json' # Jenkins updates file

  # list of jenkins plugins to be installed on the jenkins box
  plugins: [
    { name: 'ruby-runtime', version: '0.12'},
    { name: 'antisamy-markup-formatter', version: '1.3'},
    { name: 'github-api', version: '1.59'},
    { name: 'ansicolor', version: '0.4.0'},
    { name: 'external-monitor-job', version: '1.4'},
    { name: 'build-with-parameters', version: '1.3'},
    { name: 'pam-auth', version: '1.2'},
    { name: 'delivery-pipeline-plugin', version: '0.8.7'},
    { name: 'mailer', version: '1.12'},
    { name: 'junit', version: '1.3'},
    { name: 'locks-and-latches', version: '0.6'},
    { name: 'cvs', version: '2.12'},
    { name: 'github', version: '1.10'},
    { name: 'ldap', version: '1.11'},
    { name: 'jquery', version: '1.7.2-1'},
    { name: 'windows-slaves', version: '1.0'},
    { name: 'timestamper', version: '1.5.14'},
    { name: 'mapdb-api', version: '1.0.6.0'},
    { name: 'config-autorefresh-plugin', version: '1.0'},
    { name: 'ant', version: '1.2'},
    { name: 'publish-over-ssh', version: '1.12'},
    { name: 'scm-api', version: '0.2'},
    { name: 'multiple-scms', version: '0.3'},
    { name: 'buildgraph-view', version: '1.1.1'},
    { name: 'ssh-credentials', version: '1.10'},
    { name: 'log-parser', version: '1.0.8'},
    { name: 'show-build-parameters', version: '1.0'},
    { name: 'ci-game', version: '1.20'},
    { name: 'naginator', version: '1.13'},
    { name: 'jobConfigHistory', version: '2.10'},
    { name: 'javadoc', version: '1.3'},
    { name: 'throttle-concurrents', version: '1.8.4'},
    { name: 'build-flow-plugin', version: '0.17'},
    { name: 'copyartifact', version: '1.32.1'},
    { name: 'mask-passwords', version: '2.7.2'},
    { name: 'token-macro', version: '1.10'},
    { name: 'envinject', version: '1.90'},
    { name: 'analysis-core', version: '1.65'},
    { name: 'flexible-publish', version: '0.13'},
    { name: 'greenballs', version: '1.14'},
    { name: 'build-pipeline-plugin', version: '1.4.5'},
    { name: 'maven-plugin', version: '2.8'},
    { name: 'ssh-slaves', version: '1.9'},
    { name: 'matrix-project', version: '1.4'},
    { name: 'git', version: '2.3.1'},
    { name: 'git-client', version: '1.12.0'},
    { name: 'credentials', version: '1.18'},
    { name: 'gitlab-hook', version: '1.1.0'},
    { name: 'matrix-auth', version: '1.2'},
    { name: 'run-condition', version: '1.0'},
    { name: 'ssh-agent', version: '1.5'},
    { name: 'github-oauth', version: '0.20'},
    { name: 'rebuild', version: '1.22'},
    { name: 'configurationslicing', version: '1.40'},
    { name: 'parameterized-trigger', version: '2.25'},
    { name: 'build-timeout', version: '1.14'},
    { name: 'job-dsl', version: '1.26'},
    { name: 'subversion', version: '2.4.5'},
    { name: 'job-log-logger-plugin', version: '1.0'},
    { name: 'translation', version: '1.12'} ]

  jobs:
    jinja2_base_template:
      options: &base_template_options { disabled: false,
                                        concurrentbuild: false }

      buildWrappers: &base_template_wrappers
        - &BuildTimeOutWrapper_defaults { type: 'BuildTimeoutWrapper',
                                         timeoutMinutes: 60,
                                         strategy: 'AbsoluteTimeOutStrategy',
                                         failBuild: true,
                                         writingDescription: false }

        - &TimeStamper_defaults { type: 'Timestamper',
                                 options: none }

        - &AnsiColor_defaults { type: 'AnsiColor',
                               colorMapName: xterm }

    jinja2_run_ansible: &jinja2_run_ansible
      description: "Executes Ansible"
      options: { disabled: false,
                 blocks: [ 'downstream', 'upstream' ],
                 concurrentbuild: true }
      parameters: &jinja2_run_ansible_parameters
        - &inventory_file { name: 'INVENTORY_FILE',
            type: 'choice',
            description: 'Which Inventory File to use',
            choices: { type_string: ['vagrant', 'dev', 'qa', 'prd']}}

        - &playbook { name: 'PLAYBOOK',
            type: 'choice',
            description: 'Which playbook to execute',
            choices: { type_string: ['jenkins.yml', 'zabbix.yml', 'site.yml']}}

        - &limit { name: "LIMIT",
            type: 'string',
            description: "Ansible --limit",
            default: "all" }

        - &tags { name: "TAGS",
            type: 'string',
            description: "Ansible --tags",
            default: "" }

        - &start_at_task { name: "START_AT_TASK",
            type: 'string',
            description: "Ansible --start-at-task",
            default: "" }

        - &release { name: "RELEASE",
            type: 'string',
            description: "RELEASE number to use",
            default: "latest" }

        - &vault { name: "VAULT",
            type: 'password',
            description: "Ansible Vault Password",
            default: "" }

      scm:
        - { type: 'git',
            url: 'https://github.com/Azulinho/ansible-jenkins-showcase.git',
            branches: ['*/master'] }

      builders:
        - { type: 'shell',
            command_lines: [
              "#!/bin/bash",
              "export PATH=/usr/local/bin:$PATH",
              "export PYTHONUNBUFFERED=1",
              "echo $VAULT &gt; .vault",
              "ANSIBLE_HOST_KEY_CHECKING=False ansible-playbook -u vagrant -s -i $INVENTORY_FILE -l $LIMIT $PLAYBOOK --vault-password-file .vault" ]}

      buildWrappers: *base_template_wrappers

    jinja2_deploy_zabbix:
      <<: *jinja2_run_ansible
      parameters: [
        <<: *limit,
        <<: *tags,
        <<: *start_at_task,
        <<: *release,
        <<: *vault,
        { name: 'PLAYBOOK',
            type: 'string',
            description: 'Which playbook to execute',
            default: 'zabbix.yml'},
        { name: 'INVENTORY_FILE',
            type: 'string',
            description: 'Which Inventory File to use',
            default: 'vagrant'} ]
      publishers:
        - { type: 'parametrizedTrigger',
            projects: ['jinja2_deploy_zabbix_checks'],
            parameters: ['<hudson.plugins.parameterizedtrigger.CurrentBuildParameters/>'],
            condition: 'success'}

    jinja2_deploy_zabbix_checks:
      <<: *jinja2_run_ansible
      parameters: [
        <<: *limit,
        <<: *tags,
        <<: *start_at_task,
        <<: *release,
        <<: *vault,
        { name: 'PLAYBOOK',
            type: 'string',
            description: 'Which playbook to execute',
            default: 'zabbix-checks.yml'},
        { name: 'INVENTORY_FILE',
            type: 'string',
            description: 'Which Inventory File to use',
            default: 'vagrant'} ]
      publishers:
        - { type: 'parametrizedTrigger',
            projects: ['jinja2_run_zabbix_tests'],
            parameters: ['<hudson.plugins.parameterizedtrigger.CurrentBuildParameters/>'],
            condition: 'success'}

    jinja2_run_zabbix_tests:
      <<: *jinja2_run_ansible
      builders:
        - { type: 'shell',
            command_lines: [
              "#!/bin/bash",
              "echo SUCESS" ]}
      options: *base_template_options
      buildWrappers: *base_template_wrappers

    jinja2_deploy_template:
      options: &deploy_template_options { disabled: false,
                                          blocks: [ 'downstream',
                                                    'upstream' ],
                                          concurrentbuild: true }
      builders:
        - { type: 'shell',
            command_lines: [
              "#!/bin/bash",
              "echo $VAULT &gt; .vault",
              "ansible-playbook -s -i $INVENTORY_FILE -l $LIMIT $PLAYBOOK --vault-password-file .vault" ]}

    jinja2_example1:
      options: *deploy_template_options
      parameters:
        - { name: "PARAMETER1",
            type: 'string',
            description: "PARAMETER 1",
            default: "all" }
      builders:
        - { type: 'shell',
            command_lines: [
              "#!/bin/bash",
              "echo deploy_job1" ]}
      buildWrappers: *base_template_wrappers
      publishers:
        - { type: 'parametrizedTrigger', projects: ['deploy_job2'], condition: 'success', parameters: ['VAR1=var1', 'VAR2=var2'], triggerWithNoParameters: false }

    jinja2_example2:
      options: *deploy_template_options
      builders:
        - { type: 'shell',
            command_lines: [
              "#!/bin/bash",
              "echo deploy_job2" ]}
      buildWrappers: [ *BuildTimeOutWrapper_defaults, *TimeStamper_defaults, *AnsiColor_defaults ]
      publishers:
        - { type: 'parametrizedTrigger', projects: ['deploy_job3'], condition: 'success', parameters: ['VAR1=var1', 'VAR2=var2'], triggerWithNoParameters: false }

    jinja2_example3: { options: *deploy_template_options, builders: [ { type: 'shell', command_lines: [ "#!/bin/bash", "echo deploy_job3" ]} ], buildWrappers: *base_template_wrappers , publishers: [ {type: 'parametrizedTrigger', projects: ['deploy_job4']} ] }
    jinja2_example4: { options: *deploy_template_options, builders: [ { type: 'shell', command_lines: [ "#!/bin/bash", "echo deploy_job4" ]} ], buildWrappers: *base_template_wrappers , publishers: [ {type: 'parametrizedTrigger', projects: ['deploy_job5']} ]}
    jinja2_example5: { options: *deploy_template_options, builders: [ { type: 'shell', command_lines: [ "#!/bin/bash", "echo deploy_job5" ]} ], buildWrappers: *base_template_wrappers , publishers: [ {type: 'parametrizedTrigger', projects: ['deploy_job6']} ]}
    jinja2_example6: { options: *deploy_template_options, builders: [ { type: 'shell', command_lines: [ "#!/bin/bash", "echo deploy_job6" ]} ], buildWrappers: *base_template_wrappers }

views:
  list:
    - { name: 'All',
        description: 'All',
        includeRegex: '.*',
        columns: &all_columns_view [
          'hudson.views.StatusColumn',
          'hudson.views.WeatherColumn',
          'hudson.views.JobColumn',
          'hudson.views.LastSuccessColumn',
          'hudson.views.LastFailureColumn',
          'hudson.views.LastDurationColumn',
          'hudson.views.BuildButtonColumn']}

    - { name: 'DSL_BUILD',
        description: 'All BUILD jobs built using the DSL',
        includeRegex: 'DSL_BUILD.*',
        columns: *all_columns_view }

    - { name: 'DSL_DEPLOY',
        description: 'All DEPLOY jobs built using the DSL',
        includeRegex: 'DSL_DEPLOY.*',
        columns: *all_columns_view }

  # the pipeline block is used to configure the PIPELINE views
  # in jenkins.
  pipeline:
    - { name: 'PIPELINE1',
        selectedJob: 'DSL_DEPLOY-job1',
        firstJob: 'DSL_DEPLOY-job1',
        noOfDisplayedBuilds: '5',
        buildViewTitle: 'Deployment Pipeline 1' }

    - { name: 'PIPELINE2',
        selectedJob: 'jinja2_deploy_zabbix',
        firstJob: 'jinja2_deploy_zabbix',
        noOfDisplayedBuilds: '5',
        startsWithParameters: true,
        buildViewTitle: 'Zabbix Deployment Pipeline' }



