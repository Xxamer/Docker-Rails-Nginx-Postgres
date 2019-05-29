FROM ruby:2.5.1-alpine
RUN apk add --update build-base postgresql-dev tzdata nodejs 
RUN gem install bundler -v "~>2.0"


# Set an environment variable where the Rails app is installed to inside of Docker image
ENV RAILS_ROOT /home/christian/Sites/testapp 
RUN mkdir -p $RAILS_ROOT 
# Set working directory
WORKDIR $RAILS_ROOT
# Setting env up
ENV RAILS_ENV='production'
ENV RACK_ENV='production' 
# Adding gems
COPY Gemfile Gemfile
COPY Gemfile.lock Gemfile.lock
RUN bundle install --without development test 
# Adding project files
COPY . .
RUN bundle exec rake assets:precompile
EXPOSE 3000
CMD [ "bundle", "exec", "puma", "-C", "config/puma.rb" ]