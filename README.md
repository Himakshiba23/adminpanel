# Tasks

# 1) Admin panel
reference link:
https://activeadmin.info/documentation.html

steps:

In Gemfile

    gem 'activeadmin'

In terminal

   $  bundle install
   $  rails generate active_admin:install
   $  rails db:migrate
   $  rails server

In terminal

    $  rails c
    >  AdminUser.create!(:email => 'emailid@example.com', :password => 'admin123', :password_confirmation => 'admin123')

Visit http://localhost:3000/admin and log in using:
  
    User: emailid@example.com
    Password: admin123

In terminal
  
    $  rails generate active_admin:resource User
    $  rails generate active_admin:resource Book



# 2) Create New book from admin panel

reference link :
https://activeadmin.info/10-custom-pages.html

uncomment in  /app/admin/books.rb
  
    permit_params :name

-----------------------

# 3) Create New user from admin panel
add in /app/admin/users.rb
  
    permit_params :email, :name, :password, :password_confirmation 
    
    index do
      column :name
      column :email
      actions
    end

    form do |f|
      f.inputs 'User' do
        f.input :name
        f.input :email
        f.input :password
        f.input :password_confirmation
      end
  
      f.actions
    end
  
-------------------------

# 4) Pagination

reference video and link: 
http://railscasts.com/episodes/254-pagination-with-kaminari?autoplay=true
https://github.com/amatsuda/kaminari

---------------------------

# 5) Pagination with ajax

reference link:
https://rails.devcamp.com/trails/ruby-gem-walkthroughs/campsites/view-template-tools/guides/kaminari-pagination-example

in books_controller.rb
  
    def index
      @books = Book.page(params[:page]).per(10)
      respond_to do |format|
        format.js
        format.html
      end
    end

create index.js.erb
  
    $('#books').html('<%= escape_javascript render(@books) %>');
    $('#paginator').html('<%= escape_javascript(paginate(@books, remote: true).to_s) %>');

index.html.erb add remote: :true
    
    <div id="paginator">
      <%= paginate @books, remote: true %>
    </div>

--------------------------

# 6) Export CSV

At the top of your confic/application.rb file, add 
  
    require 'csv'

in books_controller.rb
  
    def index
      @books = Book.page(params[:page]).per(10)
      respond_to do |format|
        format.js
        format.html
        format.csv { send_data @books.to_csv }
      end
    end

in book.rb

    class Book < ApplicationRecord
      def self.to_csv(options = {})
        CSV.generate(options) do |csv|
          csv << column_names
          all.each do |book|
            csv << book.attributes.values_at(*column_names)
          end
        end
      end
    end
  
in index.html.erb
    
    <%= link_to "Export CSV", books_path(format: "csv"), style:"float: right" %>

--------------------------

# 7) import CSV and insert Record

in config/routes.rb
  
    resources :books do
      collection { post :import }
    end

in books_controller.rb
  
    def import
      Book.import(params[:file])
      redirect_to root_url, notice: "Books imported."
    end

in book.rb
  
    def self.import(file)
      CSV.foreach(file.path, headers: true) do |row|
        Book.create! row.to_hash
      end
    end

in index.html.erb
  
    <%= form_tag import_books_path, multipart: true do %>
      <%= file_field_tag :file %>
      <%= submit_tag "Import CSV" %>
    <% end %>

-----------------------------