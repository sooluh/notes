---
title: "Customizing User Profile and Password Change Pages in Filament 3"
datePublished: Sat Jan 20 2024 07:24:43 GMT+0000 (Coordinated Universal Time)
cuid: clrlqv6hf000109jwhk8t0a68
slug: profile-page-filament-3
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1705735411879/fc91cbde-b8d0-4222-83f3-849aaeb7c6e3.jpeg
tags: laravel, filamentphp

---

A few months ago, I started using Laravel, which I used to dislike. I discovered Filament, a form builder built with the [TALL](https://tallstack.dev) (Tailwind, Alpine.js, Laravel, Livewire) stack. I fell in love with it because it significantly sped up tasks that used to take me weeks, now completed in just a few days.

I won't go into explaining what Filament is, as there are already numerous articles about it. Instead, I'll share a case where my client asked me to create a profile and password change page using Filament.

How do you create a custom page for that? I began by making a custom page in Filament with the following command.

```bash
php artisan make:filament-page App/Profile --type=custom
```

Just hit \[Enter\] when prompted to enter a resource. With this code, it creates two new files, `app/Filament/Pages/App/Profile.php` and `resources/views/filament/pages/app/profile.blade.php`. Modify the Filament Profile page as follows.

```php
<?php

namespace App\Filament\Pages\App;

use Filament\Pages\Page;
use Illuminate\Support\Facades\Auth;

class Profile extends Page
{
    protected static string $view = 'filament.pages.app.profile';

    protected static bool $shouldRegisterNavigation = false;

    protected static ?string $title = 'Update Profile';

    protected function getViewData(): array
    {
        return [
            'user' => Auth::user(),
        ];
    }
}
```

Set the `$shouldRegisterNavigation` property to false so it doesn't appear in the navigation menu. Fill the `$title` property with the page's title. I created the `getViewData()` method, returning an array with the "user" key containing the currently logged-in user's data.

Before proceeding, ensure that the "avatar" column exists in your "users" table. You can add it directly to your user migration file or create a new migration and alter the users table to add the avatar column.

```php
$table->string('avatar')->nullable();
```

Now, create a new directory named "Traits" in the app directory and add a new file named `Avatarable.php`. Add the following code,

```php
<?php

namespace App/Traits;

use Illuminate\Database\Eloquent\Casts\Attribute;
use Illuminate\Http\UploadedFile;
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\Storage;

trait Avatarable
{
    public function updateAvatar(UploadedFile $photo): void
    {
        tap($this->avatar, function ($prev) use ($photo) {
            $this->forceFill([
                'avatar' => $photo->storePublicly('avatar', ['disk' => 'public']),
            ])->save();

            if ($prev) {
                Storage::disk('public')->delete($prev);
            }
        });
    }

    public function deleteAvatar(): void
    {
        Storage::disk('public')->delete($this->avatar);
        $this->forceFill(['avatar' => null])->save();
    }

    public function avatarUrl(): Attribute
    {
        return Attribute::get(function () {
            /** @var \Illuminate\Filesystem\FilesystemManager $disk */
            $disk = Storage::disk('public');

            return $this->avatar ? $disk->url($this->avatar) : $this->defaultAvatarUrl();
        });
    }

    protected function defaultAvatarUrl(): string
    {
        return asset('images/default-avatar.png');
    }
}
```

Then open the User model, implement `HasAvatar`, use `Avatarable`, and create a new method, `getFilamentAvatarUrl()`, something like this.

```php
<?php

namespace App\Models;

use App\Traits\Avatarable;
use Filament\Models\Contracts\HasAvatar;
// ...

class User extends Authenticatable implements HasAvatar
{
    use Avatarable;

    // ...

    public function getFilamentAvatarUrl(): ?string
    {
        return $this->avatar_url;
    }
}
```

Next, since we're not displaying the profile edit page in the navigation menu, add it to the user menu (usually in the top-right corner when clicked). Add the following code to the Filament provider, typically in `app/Providers/Filament/AdminPanelProvider.php`.

```php
<?php

namespace App\Providers\Filament;

use App\Filament\Pages\User\Profile;
use Filament\Navigation\MenuItem;
use Filament\Panel;
use Filament\PanelProvider;
// ...

class AdminPanelProvider extends PanelProvider
{
    public function panel(Panel $panel): Panel
    {
        return $panel
            // ...
            ->userMenuItems([
                'profile' => MenuItem::make()
                    ->label('Profile')
                    ->icon('heroicon-o-user-circle')
                    ->url(static fn (): string => route(Profile::getRouteName(panel: 'admin'))),
            ]);
    }
}
```

The profile menu is now visible, and the page can be accessed. I need to create 2 Livewire components for this, Profile and Password.

```bash
php artisan make:livewire App/User/Profile
php artisan make:livewire App/User/Password
```

These commands create 4 new files that you can view after running the commands. Why these 4 files? To create custom views for profile and password modification forms.

Open `app/Livewire/App/User/Password.php` and modify the code as follows.

```php
<?php

namespace App\Livewire\App\User;

use Filament\Forms;
use Filament\Forms\Concerns\InteractsWithForms;
use Filament\Forms\Contracts\HasForms;
use Filament\Forms\Form;
use Filament\Notifications\Notification;
use Illuminate\Contracts\Auth\Authenticatable;
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\Hash;
use Livewire\Component;

class Password extends Component implements HasForms
{
    use InteractsWithForms;

    public $state = [
        'current_password' => '',
        'password' => '',
        'password_confirmation' => '',
    ];

    public function form(Form $form): Form
    {
        return $form->schema([
            Forms\Components\TextInput::make('current_password')
                ->label('Recent Password')
                ->password()
                ->rules(['current_password:web'])
                ->required(),

            Forms\Components\TextInput::make('password')
                ->label('New Password')
                ->password()
                ->regex('/^(?=.*[A-Z])(?=.*[a-z])(?=.*\d)(?=.*[^\w\s]).*$/i')
                ->minLength(8)
                ->required()
                ->same('password_confirmation'),

            Forms\Components\TextInput::make('password_confirmation')
                ->label('Confirm New Password')
                ->password()
                ->regex('/^(?=.*[A-Z])(?=.*[a-z])(?=.*\d)(?=.*[^\w\s]).*$/i')
                ->minLength(8)
                ->required()
                ->same('password'),
        ])->statePath('state');
    }

    public function save(): void
    {
        $this->resetErrorBag();
        $this->validate();

        if (session() !== null) {
            session()->put([
                'password_hash_'.Auth::getDefaultDriver() => Auth::user()?->getAuthPassword(),
            ]);
        }

        /** @var \App\Model\User $user */
        $user = Auth::user();
        $user->forceFill(['password' => Hash::make($this->state['password'])])->save();

        $this->state = [
            'current_password' => '',
            'password' => '',
            'password_confirmation' => '',
        ];

        Notification::make()->success()->title('Password changed successfully.')->send();
    }

    public function getUserProperty(): ?Authenticatable
    {
        return Auth::user();
    }

    public function render()
    {
        return view('livewire.app.user.password');
    }
}
```

I hope the code is easy to understand. Next, open `app/Livewire/App/User/Profile.php` and modify it as follows.

```php
<?php

namespace App\Livewire\App\User;

use App\Filament\Pages\User\Profile as ProfilePage;
use App\Models\User;
use Filament\Forms;
use Filament\Forms\Concerns\InteractsWithForms;
use Filament\Forms\Contracts\HasForms;
use Filament\Forms\Form;
use Filament\Notifications\Notification;
use Illuminate\Contracts\Auth\Authenticatable;
use Illuminate\Support\Facades\Auth;
use Illuminate\Validation\Rules\Unique;
use Livewire\Component;
use Livewire\WithFileUploads;

class Profile extends Component implements HasForms
{
    use InteractsWithForms;
    use WithFileUploads;

    public ?array $state = [];

    public $photo;

    /** @var \App\Model\User */
    public $user;

    public function mount(): void
    {
        $this->user = Auth::user();
        $this->state = $this->user?->withoutRelations()->toArray();
    }

    public function form(Form $form): Form
    {
        return $form->schema([
            Forms\Components\TextInput::make('login')
                ->label('Username')
                ->regex('/^[a-z0-9_-]{3,15}$/i')
                ->unique(User::class, 'login', modifyRuleUsing: function (Unique $rule) {
                    return $rule->whereNot('id', auth()->user()->id);
                })
                ->required()
                ->disabledOn('edit'),

            Forms\Components\TextInput::make('email')
                ->label('Email Address')
                ->email()
                ->unique(User::class, 'email', modifyRuleUsing: function (Unique $rule) {
                    return $rule->whereNot('id', auth()->user()->id);
                })
                ->required()
                ->maxLength(255),

            Forms\Components\TextInput::make('name')
                ->label('Full Name')
                ->required()
                ->maxLength(255),
        ])->statePath('state');
    }

    public function save(): void
    {
        $this->resetErrorBag();
        $this->validate();

        if (isset($this->photo)) {
            $this->user->updateAvatar($this->photo);
        }

        $this->user->forceFill([
            'login' => $this->state['login'],
            'email' => $this->state['email'],
            'name' => $this->state['name'],
        ])->save();

        if (isset($this->photo)) {
            redirect(ProfilePage::getUrl());
        }

        Notification::make()->success()->title('Profile changed successfully.')->send();
    }

    public function deleteAvatar(): void
    {
        $this->user?->deleteAvatar();
        redirect(ProfilePage::getUrl());
    }

    public function getUserProperty(): ?Authenticatable
    {
        return Auth::user();
    }

    public function render()
    {
        return view('livewire.app.user.profile');
    }
}
```

Before changing the blade component, create a new file at `resources/views/components/app/input-error.blade.php` and add the following.

```xml
@props(['for'])

@error($for)
    <p {{ $attributes->merge(['class' => 'text-sm text-danger-600 dark:text-danger-400']) }}>
        {{ $message }}
    </p>
@enderror
```

This blade component is for error message display. For `filament/pages/app/profile.blade.php`, modify it as below, including the two Livewire components we created earlier.

```xml
<x-filament-panels::page>
    <x-filament::grid @class(['gap-6']) xl="2">
        <x-filament::grid.column>
            <livewire:app.user.profile />
        </x-filament::grid.column>

        <x-filament::grid.column>
            <livewire:app.user.password />
        </x-filament::grid.column>
    </x-filament::grid>
</x-filament-panels::page>
```

Next, open and modify the Livewire component `password.blade.php` with the following code.

```xml
<x-filament::section>
    <x-slot name="heading">
        Change Password
    </x-slot>

    <x-filament-panels::form wire:submit="save">
        {{ $this->form }}

        <div class="text-left">
            <x-filament::button type="submit">
                Save
            </x-filament::button>
        </div>
    </x-filament-panels::form>
</x-filament::section>
```

And finally, for the Livewire component `profile.blade.php`, do the following.

```xml
<x-filament::section>
    <x-slot name="heading">
        Profile Detail
    </x-slot>

    <x-filament-panels::form wire:submit="save">
        <div x-data="{ photoName: null, photoPreview: null }" class="space-y-2">
            <input type="file" class="hidden" wire:model.live="photo" x-ref="photo"
                x-on:change="
                    photoName = $refs.photo.files[0].name;
                    const reader = new FileReader();
                    reader.onload = (e) => {
                        photoPreview = e.target.result;
                    };
                    reader.readAsDataURL($refs.photo.files[0]);
            " />

            <x-filament-forms::field-wrapper.label for="photo" class="!mt-0">
                Avatar
            </x-filament-forms::field-wrapper.label>

            <x-filament::grid @class(['gap-4 items-center']) default="2" style="grid-template-columns: auto 1fr;">
                <x-filament::grid.column>
                    <div x-show="! photoPreview">
                        <x-filament-panels::avatar.user style="height: 5rem; width: 5rem;" />
                    </div>

                    <template x-if="photoPreview">
                        <img :src="photoPreview"
                            style="height: 5rem; width: 5rem; border-radius: 9999px; object-fit: cover;">
                    </template>
                </x-filament::grid.column>

                <x-filament::grid.column>
                    <x-filament::button size="sm" x-on:click.prevent="$refs.photo.click()">
                        New Avatar
                    </x-filament::button>

                    @if (isset($this->user->avatar))
                        <x-filament::button size="sm" color="danger" wire:click="deleteAvatar">
                            Remove
                        </x-filament::button>
                    @endif
                </x-filament::grid.column>
            </x-filament::grid>

            <x-app.input-error for="photo" />
        </div>

        {{ $this->form }}

        <div class="text-left">
            <x-filament::button type="submit" wire:target="photo">
                Save
            </x-filament::button>
        </div>
    </x-filament-panels::form>
</x-filament::section>
```

Done! Refresh the profile edit page, and you've successfully added a custom page to modify the profile and password. I hope readers can understand the code, as I honestly can't explain it in detail.

Hope this helps, thank you!