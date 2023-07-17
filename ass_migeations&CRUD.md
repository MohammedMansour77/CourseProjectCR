```diff
!## Course Project (Google Classroom Clone) ##

- Database Name: courseProject
- Tables Name:
    1. Classroom
    2. Topic
    3. Classwork
    4. User
    5. Submission

# Task(1): Create a draft of all required migration files based on the project's database analysis.
- Step 1: Create the migration files{
+ php artisan make:migration create_classrooms_table --create=classrooms
+ php artisan make:migration create_topics_table --create=topics
+ php artisan make:migration create_classworks_table --create=classworks
+ php artisan make:migration create_users_table --create=users
+ php artisan make:migration create_submissions_table --create=submissions
}

- database/migrations files {

+ create_classrooms_table.php :
! ### classrooms table: stores the classwork assignments of the classroom:
<?php

use Illuminate\\Database\\Migrations\\Migration;
use Illuminate\\Database\\Schema\\Blueprint;
use Illuminate\\Support\\Facades\\Schema;

class CreateClassroomTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('classroom', function (Blueprint $table) {
            $table->id(); // primary key
            $table->string('name'); // name of the classroom
            $table->unsignedBigInteger('teacher_id'); // foreign key to user table
            $table->timestamps(); // created_at and updated_at columns
            $table->foreign('teacher_id')->references('id')->on('user'); // foreign key constraint
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::dropIfExists('classroom');
    }
}

----------------------------------------------------------------
+ create_Classwork_table.php :
! ### Classwork table:
<?php

use Illuminate\\Database\\Migrations\\Migration;
use Illuminate\\Database\\Schema\\Blueprint;
use Illuminate\\Support\\Facades\\Schema;

class CreateClassworkTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('classwork', function (Blueprint $table) {
            $table->id(); // primary key
            $table->string('title'); // title of the classwork
            $table->text('description'); // description of the classwork
            $table->unsignedBigInteger('topic_id'); // foreign key to topic table
            $table->timestamps(); // created_at and updated_at columns
            $table->foreign('topic_id')->references('id')->on('topic'); // foreign key constraint
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::dropIfExists('classwork');
    }
}
----------------------------------------------------------------
+ create_topics_table.php :
!### Topic table: stores the topics of the classroom:
<?php

use Illuminate\\Database\\Migrations\\Migration;
use Illuminate\\Database\\Schema\\Blueprint;
use Illuminate\\Support\\Facades\\Schema;

class CreateTopicTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('topic', function (Blueprint $table) {
            $table->id(); // primary key
            $table->string('name'); // name of the topic
            $table->unsignedBigInteger('classroom_id'); // foreign key to classroom table
            $table->timestamps(); // created_at and updated_at columns
            $table->foreign('classroom_id')->references('id')->on('classroom'); // foreign key constraint
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::dropIfExists('topic');
    }
}

----------------------------------------------------------------
+ create_users_table.php:
! ### User table: stores the users of the application:
<?php

use Illuminate\\Database\\Migrations\\Migration;
use Illuminate\\Database\\Schema\\Blueprint;
use Illuminate\\Support\\Facades\\Schema;

class CreateUserTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('user', function (Blueprint $table) {
            $table->id(); // primary key
            $table->string('name'); // name of the user
            $table->string('email')->unique(); // email of the user, must be unique
            $table->string('password'); // password of the user, hashed by bcrypt or similar algorithm
            $table->enum('role', ['teacher', 'student']); // role of the user, either teacher or student
            $table->rememberToken(); // token for remember me feature, optional but recommended
            $table->timestamps(); // created_at and updated_at columns
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::dropIfExists('user');
    }
}

----------------------------------------------------------------
+ create_submissions_table.php
! ###: Submission table: stores the submissions of the classwork by the students
<?php

use Illuminate\\Database\\Migrations\\Migration;
use Illuminate\\Database\\Schema\\Blueprint;
use Illuminate\\Support\\Facades\\Schema;

class CreateSubmissionTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('submission', function (Blueprint $table) {
            $table->id(); // primary key
            $table->text('content'); // content of the submission, can be text or file link or anything else depending on your application logic
            $table->unsignedBigInteger('classwork_id'); // foreign key to classwork table
            $table->unsignedBigInteger('student_id'); // foreign key to user table, must be a student role user
            $table->timestamps(); // created_at and updated_at columns
            $table->foreign('classwork_id')->references('id')->on('classwork'); // foreign key constraint
            $table->foreign('student_id')->references('id')->on('user'); // foreign key constraint, you may also want to add a check constraint to ensure that student_id is a student role user, see https://laravel.com/docs/9.x/migrations#creating-columns
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::dropIfExists('submission');
    }
}

! ################################# !
}

# Task (2): Create full CRUD operation for the "Topic" resource.

{php artisan make:model Topic -mcr}

+routes/web.php file:
{
    use App\Http\Controllers\TopicController;

Route::get('/topics', [TopicController::class, 'index'])->name('topics.index');
Route::get('/topics/create', [TopicController::class, 'create'])->name('topics.create');
Route::post('/topics', [TopicController::class, 'store'])->name('topics.store');
Route::get('/topics/{topic}', [TopicController::class, 'show'])->name('topics.show');
Route::get('/topics/{topic}/edit', [TopicController::class, 'edit'])->name('topics.edit');
Route::put('/topics/{topic}', [TopicController::class, 'update'])->name('topics.update');
Route::delete('/topics/{topic}', [TopicController::class, 'destroy'])->name('topics.destroy');

}

+ TopicController.php file:
{
    namespace App\Http\Controllers;

use App\Models\Topic;
use Illuminate\Http\Request;

class TopicController extends Controller
{
    public function index()
    {
        $topics = Topic::all();
        return view('topics.index', compact('topics'));
    }

    public function create()
    {
        return view('topics.create');
    }

    public function store(Request $request)
    {
        $topic = new Topic();
        $topic->name = $request->input('name');
        // Set other fields as required
        $topic->save();

        return redirect()->route('topics.index');
    }

    public function show(Topic $topic)
    {
        return view('topics.show', compact('topic'));
    }

    public function edit(Topic $topic)
    {
        return view('topics.edit', compact('topic'));
    }

    public function update(Request $request, Topic $topic)
    {
        $topic->name = $request->input('name');
        // Update other fields as required
        $topic->save();

        return redirect()->route('topics.index');
    }

    public function destroy(Topic $topic)
    {
        $topic->delete();

        return redirect()->route('topics.index');
    }
}

}


