# WordGuesser on Rails

In a [previous assignment](https://github.com/saasbook/hw-sinatra-saas-wordguesser) you created a simple Web app that plays the Wordguesser game.

More specifically:

1. You wrote the app's code in its own class, `WordGuesserGame`, which knows nothing about being part of a Web app.

2. You used the Sinatra framework to "wrap" the game code by providing a set of RESTful actions that the player can take, with the following routes:

    * `GET  /new`-- default ("home") screen that allows player to start new game
    * `POST /create` -- actually creates the new game
    * `GET  /show` -- show current game status and let player enter a move
    * `POST /guess` -- player submits a letter guess
    * `GET  /win`   -- redirected here when `show` action detects game won
    * `GET  /lose`  -- redirected here when `show` action detects game lost

3. To maintain the state of the game between (stateless) HTTP requests, you stored a copy of the `WordGuesserGame` instance itself in the `session[]` hash provided by Sinatra, which is an abstraction for storing information in cookies passed back and forth between the app and the player's browser.

In this assignment, you'll reuse the *same* game code but "wrap" it in a simple Rails app instead of Sinatra.

## Learning Goals

Understand the differences between how Rails and Sinatra handle various aspects of constructing SaaS, including: 

* how routes are defined and mapped to actions; 
* the directory structure used by each framework;
* how an app is started and stopped; 
* how the app's behavior can be inspected by looking at logs or invoking a debugger. 

## 1. Run the App

**NOTE: You may find these [Rails guides](http://guides.rubyonrails.org/v4.2/) and the [Rails reference documentation](http://api.rubyonrails.org/v4.2.9/) helpful to have on hand.**

Like substantially all Rails apps, you can get this one running by doing these steps:

1. Clone or fork the [repo](https://github.com/saasbook/hw-rails-wordguesser)

2. Change into the app's root directory `hw-rails-wordguesser`

3. Run `bundle install --without production`

4. | Local Development                      	| Codio                                                     	|
    |----------------------------------------	|-----------------------------------------------------------	|
    | Run `rails server` to start the server 	| Run <br>`rails server -b 0.0.0.0`<br> to start the server 	|

### To view your site in Codio
Use the Preview button that says "Project Index" in the top tool bar. Click the drop down and select "Box URL" 

![.guides/img/BoxURLpreview](https://global.codio.com/content/BoxURLpreview.png)

For subsequent previews, you will not need to press the drop down -- your button should now read "Box URL".

**Q1.1.**  What is the goal of running `bundle install`?
- เป้าหมายของการรันคำสั่ง bundle install คือการติดตั้งทุก gem ที่ระบุในไฟล์ Gemfile ลงในระบบ

**Q1.2.**  Why is it good practice to specify `--without production` when running  it on your development computer?
- การใช้ --without production ช่วยลดการติดตั้ง dependencies ที่ไม่จำเป็นในโหมดการพัฒนา ซึ่งจะทำให้ระบบทำงานเร็วขึ้นและลดขนาดของการติดตั้ง

**Q1.3.** 
(For most Rails apps you'd also have to create and seed the development database, but like the Sinatra app, this app doesn't use a database at all.)

Play around with the game to convince yourself it works the same as the Sinatra version.
- สำหรับแอปนี้ ไม่จำเป็นต้องใช้ฐานข้อมูลเพราะแอปไม่ได้ใช้ฐานข้อมูลอยู่แล้ว แต่ยังสามารถรันคำสั่ง rails server ได้เหมือนกับแอป Sinatra

## 2. Where Things Are

Both apps have similar structure: the user triggers an action on a game via an HTTP request; a particular chunk of code is called to "handle" the request as appropriate; the `WordGuesserGame` class logic is called to handle the action; and usually, a view is rendered to show the result.  But the locations of the code corresponding to each of these tasks is slightly different between Sinatra and Rails.

**Q2.1.** Where in the Rails app directory structure is the code corresponding to the `WordGuesserGame` model?

- โค้ดที่เกี่ยวข้องกับโมเดล WordGuesserGame อยู่ใน models/word_guesser_game.rb

**Q2.2.** In what file is the code that most closely corresponds to the  logic in the Sinatra apps' `app.rb` file that handles incoming user actions?
- โค้ดที่เกี่ยวข้องกับการจัดการคำขอจากผู้ใช้ในแอป Sinatra จะอยู่ในไฟล์ app/controllers/game_controller.rb ของ Rails

**Q2.3.** What class contains that code?
- คลาสที่มีโค้ดนี้คือ GameController

**Q2.4.** From what other class (which is part of the Rails framework) does that class inherit? 
- คลาส GameController สืบทอดมาจาก ApplicationController ซึ่งเป็นคลาสพื้นฐานของ Rails

**Q2.5.** In what directory is the code corresponding to the Sinatra app's views (`new.erb`, `show.erb`, etc.)? 
- โค้ดที่เกี่ยวข้องกับการแสดงผลในแอป Sinatra จะอยู่ใน app/views

**Q2.6.** The filename suffixes for these views are different in Rails than they were in the Sinatra app.  What information does the rightmost suffix of the filename  (e.g.: in `foobar.abc.xyz`, the suffix `.xyz`) tell you about the file contents?  
- Suffix .erb หมายถึงไฟล์ HTML ที่มี Ruby code ฝังอยู่ภายใน ซึ่ง Rails จะประมวลผล Ruby เพื่อสร้าง HTML สำหรับการแสดงผล

**Q2.7.** What information does the  other suffix tell you about what Rails is being asked to do with the file?
- Suffix อื่น ๆ เช่น .html ระบุว่าไฟล์นั้นมีโครงสร้างและเนื้อหาของหน้าเว็บ

**Q2.8.** In what file is the information in the Rails app that maps routes (e.g. `GET /new`)  to controller actions?  
- ไฟล์ที่ใช้แม็ปเส้นทาง (routes) กับการทำงานของ controller อยู่ใน config/routes.rb

**Q2.9.** What is the role of the `:as => 'name'` option in the route declarations of `config/routes.rb`?  (Hint: look at the views.)
- ตัวเลือก :as => 'name' ใช้เพื่อกำหนดชื่อให้กับเส้นทางใน config/routes.rb ซึ่งสามารถใช้ในวิวยูเพื่อสร้างลิงก์

## 3. Session

Both apps ensure that the current game is loaded from the session before any controller action occurs, and that the (possibly modified) current game is replaced in the session after each action completes.

**Q3.1.** In the Sinatra version, `before do...end` and `after do...end` blocks are used for session management.  What is the closest equivalent in this Rails app, and in what file do we find the code that does it?
- ในเวอร์ชัน Sinatra ใช้ before do...end และ after do...end สำหรับการจัดการเซสชัน แต่ใน Rails ใช้ before_action :get_game_from_session และ after_action :store_game_in_session ซึ่งสามารถพบได้ในไฟล์ game_controller.rb

**Q3.2.** A popular serialization format for exchanging data between Web apps is [JSON](https://en.wikipedia.org/wiki/JSON).  Why wouldn't it work to use JSON instead of YAML?  (Hint: try replacing `YAML.load()` with `JSON.parse()` and `.to_yaml` with `.to_json` to do this test.  You will have to clear out your cookies associated with `localhost:3000`, or restart your browser with a new Incognito/Private Browsing window, in order to clear out the `session[]`.  Based on the error messages you get when trying to use JSON serialization, you should be able to explain why YAML serialization works in this case but JSON doesn't.)
- JSON อาจไม่ทำงานได้เพราะไม่รองรับการจัดการกับข้อมูลที่มีการอ้างอิงวงกลม (circular references) ในขณะที่ YAML รองรับการจัดการกับข้อมูลประเภทนี้


## 4. Views

**Q4.1.** In the Sinatra version, each controller action ends with either `redirect` (which as you can see becomes `redirect_to` in Rails) to redirect the player to another action, or `erb` to render a view.  Why are there no explicit calls corresponding to `erb` in the Rails version? (Hint: Based on the code in the app, can you discern the Convention-over-Configuration rule that is at work here?)
- ในเวอร์ชัน Sinatra เราต้องระบุการเรียกใช้ erb เพื่อแสดงผล แต่ใน Rails ด้วยหลัก Convention-over-Configuration ระบบจะทำการแสดงผลโดยอัตโนมัติจากชื่อของแอ็คชันในคอนโทรลเลอร์

**Q4.2.** In the Sinatra version, we directly coded an HTML form using the `<form>` tag, whereas in the Rails version we are using a Rails method `form_tag`, even though it would be perfectly legal to use raw HTML `<form>` tags in Rails.  Can you think of a reason Rails might introduce this "level of indirection"?
- การใช้ form_tag ใน Rails ช่วยให้จัดการข้อมูลฟอร์มได้สะดวกและมีประสิทธิภาพมากขึ้น

**Q4.3.** How are form elements such as text fields and buttons handled in Rails?  (Again, raw HTML would be legal, but what's the motivation behind the way Rails does it?)
- Rails ใช้ helpers เช่น text_field_tag และ button_tag เพื่อจัดการกับฟอร์มอย่างชัดเจนและสะดวd

**Q4.4.** In the Sinatra version, the `show`, `win` and `lose` views re-use the code in the `new` view that offers a button for starting a new game. What Rails mechanism allows those views to be re-used in the Rails version?  
- ใน Rails เราสามารถใช้ "Partial Views" ซึ่งช่วยให้เราสามารถใช้โค้ดส่วนที่เหมือนกันซ้ำได้โดยการใช้ชื่อไฟล์ที่เริ่มต้นด้วย _ เช่น _form.html.erb

## 5. Cucumber scenarios

The Cucumber scenarios and step definitions (everything under `features/`, including the support for Webmock in `webmock.rb`) was copied verbatim from the Sinatra version, with one exception: the `features/support/env.rb` file is simpler because the `cucumber-rails` gem automatically does some of the things we had to do explicitly in that file for the Sinatra version.

Verify the Cucumber scenarios run and pass by running `rake cucumber`.

**Q5.1.** What is a qualitative explanation for why the Cucumber scenarios and step definitions didn't need to be modified at all to work equally well with the Sinatra or Rails versions of the app?
- การที่ Cucumber scenarios และ step definitions ไม่ต้องถูกปรับแก้ใดๆ เนื่องจาก Cucumber ใช้ Gherkin language ที่มุ่งเน้นที่การทดสอบพฤติกรรมระดับสูงที่ไม่ขึ้นอยู่กับการใช้งานเทคโนโลยีหรือการติดตั้งในแอป.
