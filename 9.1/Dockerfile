FROM gitlab/gitlab-ce:9.1.9-ce.0

ENV GITLAB_VERSION=v9.1.9
RUN set -xe \
    && apt-get update \
    && apt-get install -yqq locales \
    && locale-gen en_US.UTF-8 \
    && apt-get purge -y --auto-remove \
        -o APT::AutoRemove::RecommendsImportant=false \
        -o APT::AutoRemove::SuggestsImportant=false \
        locales \
    && rm -rf /var/lib/apt/lists/*
ENV LANG=en_US.UTF-8 LANGUAGE=en_US:en LC_ALL=en_US.UTF-8
ENV TZ=Asia/Shanghai

RUN set -xe \
    && echo " # Preparing ..." \
    && curl -sL https://deb.nodesource.com/setup_6.x | bash - \
    && curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - \
    && echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list \
    && apt-get update \
    && buildDeps='lsb-release patch nodejs python build-essential yarn cmake' \
    && apt-get install -yqq $buildDeps \
    && echo " # Generating translation patch ..." \
    && cd / \
    && git clone https://gitlab.com/xhang/gitlab.git \
    && cd gitlab \
    && git diff v9.1.9..v9.1.9-zh -- . ':!spec' ':!features' ':!.gitignore' > ../zh_CN.diff \
    && echo " # Patching ..." \
    && patch -d /opt/gitlab/embedded/service/gitlab-rails -p1 < ../zh_CN.diff \
    && echo " # Regenerating the assets" \
    && cd /opt/gitlab/embedded/service/gitlab-rails \
    && cp config/gitlab.yml.example config/gitlab.yml \
    && cp config/database.yml.postgresql config/database.yml \
    && cp config/secrets.yml.example config/secrets.yml \
    && rm -rf public/assets \
    && yarn install --pure-lockfile \
    && RAILS_ENV=production \
        NO_PRIVILEGE_DROP=true \
        USE_DB=false \
        SKIP_STORAGE_VALIDATION=true \
        bundle exec rake gitlab:assets:compile \
    && rm -rf log \
        tmp \
        config/gitlab.yml \
        config/database.yml \
        config/secrets.yml \
        .secret \
        .gitlab_shell_secret \
        .gitlab_workhorse_secret \
        node_modules \
    && echo " # Cleaning ..." \
    && yarn cache clean \
    && apt-get purge -y --auto-remove \
        -o APT::AutoRemove::RecommendsImportant=false \
        -o APT::AutoRemove::SuggestsImportant=false \
        $buildDeps \
    && find /usr/lib/ -name __pycache__ | xargs rm -rf \
    && rm -rf /gitlab /*.diff /root/.cache /var/lib/apt/lists/*
