---
title: "「EverydayRails」のcontrollersのテストをrequest specで書き換える"
emoji: "😎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Rails", "Rspec"]
published: false
---

Railsに最近入門中のりゅーそうです。
RailsのテストではRspecが用いられることが多いようですね。
Rspecの使用方法から、テストの手法・考え方まで体系的に学べるものとして「[EverydayRails -RspecによるRailsテスト入門-](https://leanpub.com/everydayrailsrspec-jp)」という素晴らしい書籍があります。
この本には現在にも通じるようなテストの手法が体系的に書かれており、まさに満足といった感じです。
しかし、書かれてから多少歳月が立っているということもありバージョン等で差異が多少出てきているようです(と言っても翻訳者である伊藤さんによる[サポートページ](https://blog.jnito.com/entry/2019/10/25/053521)があるため大体のことはこれを見ればRails6以降のバージョンにも対応出来ると思います)。

書籍でも触れられていますが、RailsのControllerをテストする際にはRails4系統の時にはController specが用いられていましたが、現在ではそれらのメソッドは非奨励となりRequest specを用いたテストが奨励されているそうです。
本書でも7章でrequest specでControllerのテストを書き換える実装が紹介されています。
しかし、全てのコードをrequest specで書き換える例は見られませんでした。
そこで当記事ではController specを書き換える方法を紹介したいと思います。

# お断り
Railsの実務経験はないので、Controllerをどのようにテストすべきか(しないべきか)の明確な答えを持っていません。
単に本書のコード例をrequest specの手法に沿って書き換えた記事になります。
また実装するに当たってEverydayRailsの翻訳者である伊藤さんにアドバイスをいただきました。
伊藤さんありがとうございました。
https://teratail.com/questions/324966


# 前提条件とControllerテストについて
## 前提条件
当記事はEverydayRailsを読んでいる方を想定していますが、読んでいない方でも概要がわかるように前提条件を紹介します。
本書では、TODOアプリのようなアプリケーションにRspecでテストを書いていきます。
- Loginしたユーザーはprojectの追加・削除・更新が出来ます。
- projectにはtaskを追加・削除・更新が出来るアプリケーションです。
URLとAPIは以下のようになります。
```
project_tasks     GET    /projects/:project_id/tasks(.:format)          tasks#index
                  POST   /projects/:project_id/tasks(.:format)          tasks#create
new_project_task  GET    /projects/:project_id/tasks/new(.:format)      tasks#new
edit_project_task GET    /projects/:project_id/tasks/:id/edit(.:format) tasks#edit
project_task      GET    /projects/:project_id/tasks/:id(.:format)      tasks#show
```
またテスト用のデータを作成するためにFactoryBotを利用しています。
```ruby
#userのFactory
FactoryBot.define do
  factory :user, aliases: [:owner] do
    first_name { "Aaron" }
    last_name { "Aaron" }
    sequence(:email) { |n| "tester#{n}@example.com" }
    password { "dottle-nouveau-pavilion-tights-furze" }
  end
end
```

```ruby
#projectのFactory
FactoryBot.define do
  factory :project do
    sequence(:name) { |n| "Test Project #{n}" }
    description { "A test project." }
    due_on { 1.week.from_now }
    association :owner

    #メモ付きのプロジェクト
    trait :with_notes do
      after(:create) { |project| create_list(:note, 5, project: project) }
    end

    #昨日が締め切りのプロジェクト
    trait :due_yesterday do
      due_on { 1.day.ago }
    end

    #今日が締め切りのプロジェクト
    trait :due_today do
      due_on { Date.current.in_time_zone }
    end

    #明日が締め切りのプロジェクト
    trait :due_tomorrow do
      due_on { 1.day.from_now }
    end

    #無効になっている
    trait :invalid do
      name { nil }
    end
  end
end
```

## RailsのControllerのテスト
Rails5ではassignsとassert_templateが非奨励になりました。
https://rspec.info/ja/blog/2016/07/rspec-3-5-has-been-released/
代わりにrequest specを用いることを奨励しています。理由としては、Request specはController specと同様にControllerにフォーカスしたテストを行うことが出来る上に、ルータやミドルウェアなどにも関与することが出来るため、現実に近い環境でテストを行うことが出来るようになります。


# ControllerとRequest specの実装の差異
Controller specのテストは以下のようになります。
```ruby
require 'rails_helper'

RSpec.describe HomeController, type: :controller do 
  describe "#index" do
    it "responds successfully" do
      get :index
      expect(response).to be_success 
    end
  end 
end
```
Controller specのテストはtype::controllerというようにspecを設定することで、呼び出すコントローラーが設定されます。なので上記のように`get :index`と呼び出すことによって、Homeコントローラーのルートパスが呼び出されます。

Request specのテストを書く場合にはrails_helperに以下の設定を追加します。
```ruby
  #rails_helper.rb
  config.include Devise::Test::IntegrationHelpers, type: :request
```
Controller specとは異なり、Request specでテストを書く場合はルートを明示的に設定する必要があります。
このようにすることでAPIを適切にテストをすることが出来るようになります。
```ruby
require 'rails_helper'

RSpec.describe "Homes", type: :request do
  describe "#index" do
    #正常にレスポンスを返すこと
    it "responds successfully returns a 200 response" do
      get root_path
      expect(response).to be_successful
    end
  end
end
```

# Projectsをテストする
request specでAPIのテストを行っていきます。
まずはproject_request_spec.rbのテストです。

## GET
先ほどのテストと特に変わる部分はありません。
今回のテーマとの話からはズレますが、beforeメソッドでFactoryBotのuser情報を取得して、権限のテストを行っています。
```ruby
RSpec.describe "Projects", type: :request do
  describe "#index" do
    context "as a authenticated user" do
      before do
        @user = FactoryBot.create(:user)
      end
      it "responds successfully returns a 200 response" do
        sign_in @user
        get projects_path
        expect(response).to be_successful
        expect(response).to have_http_status "200"
      end
    end
    context "as a guest" do
      it "returns a 302 response" do
        get projects_path
        expect(response).to have_http_status "302"
      end
      it "redirects to the sign-in page" do
        get projects_path
        expect(response).to redirect_to "/users/sign_in"
      end
    end
  end
end
```

## POST

パスにparamsを追加することによって、projects/:project_idのパスを設定します。
Factorybotの値をハッシュとして渡す場合にはFactoryBot.attributes_forを使用します。
参考：https://qiita.com/__kotaro_/items/adbc355bfb550b8b2150

```ruby
RSpec.describe "Projects", type: :request do
  describe "#create" do
      context "as an authenticated user" do
        before do
          @user = FactoryBot.create(:user)
        end
        context "with valid attributes" do
          it "adds a project" do
            project_params = FactoryBot.attributes_for(:project)
            sign_in @user
            expect {
              post projects_path, params: { project: project_params }
            }.to change(@user.projects, :count).by(1)
          end
        end
        context "with invalid attributes" do
          it "does not add a project" do
            project_params = FactoryBot.attributes_for(:project, :invalid)
            sign_in @user
            expect {
              post projects_path, params: { project: project_params }
            }.to_not change(@user.projects, :count)
          end
        end
      end
    end
  end
end
```
## PUT DELETE
Request spec特有の実装は見られないので省略。
全容はこちらをご覧ください。
https://github.com/YouheiNozaki/Everyday-rails-v6/blob/main/spec/requests/projects_request_spec.rb

# Projects/taskをテストする
最後にtasks_request_spec.rbをテストします。
ここではJSON形式でAPIがレスポンスを返すかをテストしています。
このJSONなどのFormatをテストする場合にはletを使用して値を設定します。
ここではresponseのheaderをapplication/jsonに設定します。
```ruby
  let(:headers) do
    { 'Accept' => 'application/json' }
  end
```
全容は以下のようになります。
今回のポイントは
`responds with JSON formatted output`のテストです。
このtaskをPOSTするメソッドはURLが
`/projects/:project_id/tasks(.:format)`というように重曹になっているので、これをパスに設定する必要があります。
ここでは、
      `post project_tasks_url(@project), headers: headers, params: { task: new_task }`
というように設定します。POSTメソッドのheadersには先ほどletメソッドで設定した値を追加します。
```ruby
require 'rails_helper'

RSpec.describe "Tasks", type: :request do
  let(:headers) do
    { 'Accept' => 'application/json' }
  end
  before do
    @user = FactoryBot.create(:user)
    @project = FactoryBot.create(:project, owner: @user)
    @task = @project.tasks.create!(name: "Test task")
  end

  describe "#show" do
    #JSON形式でレスポンスを返すこと
    it "responds with JSON formatted output" do
      sign_in @user
      get projects_url, headers: headers, params: { project_id: @project.id, id: @task.id }
      expect(response.content_type).to eq "application/json"
    end
  end
  describe "#create" do
    #JSON形式でレスポンスを返すこと
    it "responds with JSON formatted output" do
      new_task = { name: "New test task" }
      sign_in @user
      post project_tasks_url(@project), headers: headers, params: { task: new_task }
      expect(response.content_type).to eq "application/json"
    end
  end
end
```
# 最後に
今回、ControllerのテストをRequest specに書き換えたことで、アプリケーションの不具合が発見されたとのことで取り組んで良かったと思います。
https://github.com/everydayrails/everydayrails-rspec-2017/issues/112
(issueを伊藤さんが作成してくださいました)

また、今回実装にあたって伊藤さん(https://twitter.com/jnchito)には細かい実装の方法やドキュメントの紹介まで詳しく教えていただきました。
改めてお礼申し上げます。

