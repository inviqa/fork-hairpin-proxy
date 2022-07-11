FROM ruby:2.7.6-alpine AS build

ENV GEM_HOME="/usr/local/bundle"
ENV PATH $GEM_HOME/bin:$GEM_HOME/gems/bin:$PATH
RUN echo "gem: --no-document" > ~/.gemrc

RUN apk add --no-cache \
  build-base \
  ruby-dev

COPY Gemfile Gemfile.lock /app/
WORKDIR /app/
RUN bundle install --jobs 16 --retry 5

FROM ruby:2.7.6-alpine AS run
COPY --from=build $GEM_HOME $GEM_HOME
COPY Gemfile Gemfile.lock /app/
WORKDIR /app/

COPY ./src/ /app/src/

CMD ["/app/src/main.rb"]
