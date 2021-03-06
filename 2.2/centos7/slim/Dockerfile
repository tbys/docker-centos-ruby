FROM centos:7
MAINTAINER Yuya.Nishida.

# Following codes are modified from:
# https://github.com/docker-library/ruby/blob/36c87bb64362de2e6e9ea003104030ca2a5fc4e5/2.3/stretch/slim/Dockerfile
#
# And environments from:
# https://github.com/docker-library/ruby/blob/b5ef401d348ca9b1d9f6a5cb4b25f32bf013daca/2.2/jessie/Dockerfile

RUN yum install -y \
		bzip2 \
		ca-certificates \
		libffi-devel \
		gdbm \
		openssl-devel \
		libreadline \
		libyaml-devel \
		procps-ng \
		zlib-devel

# skip installing gem documentation
RUN mkdir -p /usr/local/etc \
	&& { \
		echo 'install: --no-document'; \
		echo 'update: --no-document'; \
	} >> /usr/local/etc/gemrc

ENV RUBY_MAJOR 2.2
ENV RUBY_VERSION 2.2.10
ENV RUBY_DOWNLOAD_SHA256 bf77bcb7e6666ccae8d0882ea12b05f382f963f0a9a5285a328760c06a9ab650
ENV RUBYGEMS_VERSION 2.7.7
ENV BUNDLER_VERSION 1.16.2

# some of ruby's build scripts are written in ruby
#   we purge system ruby later to make sure our final image uses what we just built
RUN set -ex \
	\
	&& buildDeps=' \
		autoconf \
		bison \
		gcc \
		bzip2-devel \
		gdbm-devel \
		glib2-devel \
		ncurses-devel \
		libreadline-devel \
		libxml2-devel \
		libxslt-devel \
		make \
	' \
	&& yum install -y $buildDeps \
	&& curl --output ruby.tar.xz "https://cache.ruby-lang.org/pub/ruby/${RUBY_MAJOR%-rc}/ruby-$RUBY_VERSION.tar.xz" \
	&& echo "$RUBY_DOWNLOAD_SHA256 *ruby.tar.xz" | sha256sum -c - \
	\
	&& mkdir -p /usr/src/ruby \
	&& tar -xJf ruby.tar.xz -C /usr/src/ruby --strip-components=1 \
	&& rm ruby.tar.xz \
	\
	&& cd /usr/src/ruby \
	\
# hack in "ENABLE_PATH_CHECK" disabling to suppress:
#   warning: Insecure world writable dir
	&& { \
		echo '#define ENABLE_PATH_CHECK 0'; \
		echo; \
		cat file.c; \
	} > file.c.new \
	&& mv file.c.new file.c \
	\
	&& autoconf \
	&& gnuArch="$(uname -m)-linux-gnu" \
	&& ./configure \
		--build="$gnuArch" \
		--disable-install-doc \
		--enable-shared \
	&& make -j "$(nproc)" \
	&& make install \
	\
	&& yum erase -y $buildDeps \
	&& cd / \
	&& rm -r /usr/src/ruby \
	\
	&& gem update --system "$RUBYGEMS_VERSION" \
	&& gem install bundler --version "$BUNDLER_VERSION" --force \
	&& rm -r /root/.gem/

# install things globally, for great justice
# and don't create ".bundle" in all our apps
ENV GEM_HOME /usr/local/bundle
ENV BUNDLE_PATH="$GEM_HOME" \
	BUNDLE_SILENCE_ROOT_WARNING=1 \
	BUNDLE_APP_CONFIG="$GEM_HOME"
# path recommendation: https://github.com/bundler/bundler/pull/6469#issuecomment-383235438
ENV PATH $GEM_HOME/bin:$BUNDLE_PATH/gems/bin:$PATH
# adjust permissions of a few directories for running "gem install" as an arbitrary user
RUN mkdir -p "$GEM_HOME" && chmod 777 "$GEM_HOME"
# (BUNDLE_PATH = GEM_HOME, no need to mkdir/chown both)

CMD [ "irb" ]
