---
layout: post
title: "Yet another post"
description: "But really just a reference sample"
category: programming
tags: [tag one, tag two]
---
{% include JB/setup %}

Bacon ipsum dolor amet turducken beef ribs andouille tail shank, swine tenderloin. Ham hock shank turducken pastrami. Sirloin pork loin prosciutto boudin, pig shoulder andouille meatloaf ```turkey``` ground round picanha. Beef andouille jowl shankle salami pork loin rump ground round spare ribs pork chop sausage tongue pig. Pastrami pancetta meatball, cow shank swine filet mignon cupim bresaola. Pork belly bacon filet mignon cupim, bresaola jerky t-bone boudin brisket biltong shoulder ground round. Tri-tip sirloin landjaeger alcatra, pastrami shank jowl sausage pig pork short loin pork chop doner spare ribs.
<!--break-->

{% highlight ruby lineanchors %}
def create
  if params[:friendship] && params[:friendship].has_key?(:friend_id)
    @friend = User.friendly.find(params[:friendship][:friend_id])
    @friendship = Friendship.request(current_user, @friend)
    respond_to do |format|
      if @friendship.new_record?
        format.html do
          flash[:error] = "Something went wrong."
          redirect_to friendships_path
        end
        format.json { render json: @friendship.to_json, status: :precondition_failed }
      else
        format.html do
          flash[:success] = "Friend request sent."
          redirect_to friendships_path
        end
        format.json { render json: @friendship.to_json }
      end
    end

  else
    flash[:error] = "Friend required."
    redirect_to statuses_path
  end
end

{% endhighlight %}

Shank ribeye shankle ham pancetta. Doner shank short ribs prosciutto jowl t-bone swine. Drumstick chuck meatball tail strip steak meatloaf sirloin filet mignon swine flank porchetta. Ham hock bacon porchetta pastrami boudin venison cow tri-tip frankfurter meatball hamburger beef ribs flank biltong. Spare ribs chuck jowl venison rump. Flank pork belly leberkas, venison prosciutto pork sirloin. Leberkas pork chop biltong, filet mignon andouille pork pork loin swine bresaola pancetta tail chuck shankle meatloaf venison.

Bacon spare ribs tail salami, picanha meatball sirloin. Swine landjaeger cupim chuck spare ribs, andouille picanha pancetta pastrami cow. Pig tongue capicola kevin chuck strip steak meatloaf tail chicken leberkas brisket hamburger corned beef. Pig t-bone tail bacon ham tri-tip, turducken bresaola pork salami brisket spare ribs pancetta capicola.

Frankfurter ball tip biltong cupim, beef ribs venison shankle ground round chuck turducken short loin filet mignon. Shank tongue t-bone venison. Capicola jerky cow sausage. Pork ham chuck fatback.

Rump bresaola brisket, meatball biltong prosciutto shoulder andouille tongue cupim alcatra jowl flank. Sirloin rump boudin, alcatra meatball corned beef tenderloin sausage pancetta tri-tip. Filet mignon andouille beef porchetta tenderloin capicola kielbasa short ribs pork chop bacon picanha shank tail. Spare ribs tenderloin capicola chicken, boudin cupim strip steak. Bresaola shankle strip steak spare ribs meatloaf pork loin hamburger t-bone short loin.
