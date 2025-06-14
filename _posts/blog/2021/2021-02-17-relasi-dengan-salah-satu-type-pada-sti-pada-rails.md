---
layout: 'post'
title: "Membuat Relasi dengan Hanya Salah Satu Type pada Single Table Inheritance Model di Rails"
date: '2021-02-17 20:52'
permalink: '/blog/:title'
author: 'BanditHijo'
license: true
comments: true
toc: true
category: 'blog'
tags: ['Rails']
pin:
hot:
contributors: []
description: "Menyambung catatan tentang Single Table Inheritance, namun kali ini mengenai relasi salah satu type dari model yang dipasang STI."
---

# Prerequisite

`ruby 3.0.0` `rails 6.1.2` `postgresql 12.5` `rspec 4.0.0`


# Latar Belakang Masalah

Catatan kali ini, saya akan membahas Ruby on Rails dari sisi back-end.

Melanjutkan catatan mengenai Single Table Inheritance sebelumnya, yaitu tentang [**Mengenal Single Table Inheritance dengan Devise pada Rails (Contoh 2)**](/blog/rails-single-table-inheritance-dengan-devise-contoh-2).

Sekarang saya akan lanjutkan dengan relasi antar type, namun catatan kali ini mencontohkan salah satu type saja.

Misalkan saya memilik model **User** yang memiliki dua type user, yaitu **Developer** dan **Copywriter**.

**users**

|----|--------------------|------------|
| id | email              | type       |
|----|--------------------|------------|
| 1  | bayuyuba@gmail.com | Developer  |
| 2  | bololoba@gmail.com | Copywriter |
|----|--------------------|------------|

**news**

|-------|---------|---------|
| title | content | user_id |
|-------|---------|---------|
| ...   | ...     | 2       |
|-------|---------|---------|

Sudah dapat ditebak yaa, tabel news tentu memiliki relasi one to many dengan tabel users.

Karena saya menggunakan STI pada tabel Users, maka saya juga perlu membuat model untuk masing-masing type user (developer dan copywriter).

```ruby
!filename: app/models/developer.rb
class Developer < User
end
```

```ruby
!filename: app/models/copywriter.rb
class Copywriter < User
end
```

Kita akan mencoba meletakkan method association-nya pada model **User** dan juga model **News**.

```ruby
!filename: app/models/user.rb
class User < ApplicationRecord
  has_many :news, dependent: :destroy
end
```

```ruby
!filename: app/models/news.rb
class News < ApplicationRecord
  belongs_to :user
end
```

Kalau asosiasinya seperti di atas, maka user **Developer** juga dapat membuat **News**.

```ruby
irb(main)> user_developer = Developer.first
=> #<Developer id: 1, email: "bayuyuba@gmail.com", ...

irb(main)> user_developer.news
=> #<ActiveRecord::Associations::CollectionProxy []>
```

Sedangkan, yang kita inginkan adalah hanya user **Copywriter** yang dapat membuat News.

```ruby
irb(main)> user_copywriter = Copywriter.first
=> #<Copywriter id: 2, email: "bololoba@gmail.com", ...

irb(main)> user_copywriter.news
=> #<ActiveRecord::Associations::CollectionProxy []>
```

Developer juga dapat membuat News karena kita meletakkan association-nya pada model User (`has_many :news`) dan pada model News (`belongs_to :user`). Sedangkan pada model User, terdapat dua type user, yaitu Developer dan Copywriter. Tentu saja kedua type user tersebut jadi memiliki association terhadap model News.


# Pemecahan Masalah

Sudah dapat ditebak yaa, seharusnya kita meletakkan associationnya pada model **Copywriter**, bukan pada model **User**.

Maka, kita perlu menghapus association yang ada pada model **User** dan memindahkannya ke model **Copywriter**.

```ruby
!filename: app/models/copywriter.rb
class Copywriter < User
  has_many :news, dependent: :destroy, foreign_key: :user_id
end
```

Sedangkan association pada model **News** kita akan ganti objectnya ke model **Copywriter**.

```ruby
!filename: app/models/news.rb
class News < ApplicationRecord
  belongs_to :copywriter, foreign_key: :user_id
end
```

Yang menjadi kunci dari association antara model **Copywriter** dengan model **News** di atas adalah `foreign_key: :user_id`.

Sangat perlu untuk didefinisikan, karena model News sebenarnya tidak mengetahui siapa itu Copywriter. Model News hanya mengenal User, nanti model User yang akan memberitahu, bahwa yang si News cari adalah salah satu type user yang dimiliki oleh model User, yaitu Copywriter. 😅

Hal ini karena secara skematik, pada tabel News, kita merelasikan dengan tabel Users. Sedangkan Copywriter tidak memiliki tabel.

```ruby
!filename: db/schema.rb
# ...

  create_table "news", force: :cascade do |t|
    t.string   "title"
    t.bigint   "user_id",    null: false
    t.datetime "created_at", precision: 6, null: false
    t.datetime "updated_at", precision: 6, null: false
    t.index    ["user_id"],  name: "index_news_on_user_id"
  end

# ...
```


# Pengujian

Kita akan mulai dari Copywriter.

```ruby
irb(main)> user_copywriter = Copywriter.first
=> #<Copywriter id: 2, email: "bololoba@gmail.com", ...

irb(main)> user_copywriter.news
=> #<ActiveRecord::Associations::CollectionProxy []>
```

Nah, copywriter **masih** memiliki berasosiasi dengan model News.

Seharusnya model Developer, sudah tidak dapat membuat News.

```ruby
irb(main)> user_developer = Developer.first
=> #<Developer id: 1, email: "bayuyuba@gmail.com", ...

irb(main)> user_developer.news
NoMethodError (undefined method `news' for #<Developer id: 1, email: "bayuyuba@gmail.com", ...">)
```

Oke, mantap! Developer sudah **tidak dapat** membuat News.


# Pesan Penulis

Sepertinya, segini dulu yang dapat saya tuliskan.

Selanjutnya, saya serahkan kepada imajinasi dan kreatifitas teman-teman. Hehe.

Mudah-mudahan dapat bermanfaat.

Terima kasih.

(^_^)


# Referensi

1. [guides.rubyonrails.org/association_basics.html#single-table-inheritance-sti](https://guides.rubyonrails.org/association_basics.html#single-table-inheritance-sti)
<br>Diakses tanggal: 2021/02/17
