## RAC-小知识
<!--more-->
![在MVVM中，我们趋向于将view和view controller作为一个整体,
新的viewModel代替原来的viewController协调view与model之间的交互。](https://kevinhm.gitbooks.io/functionalreactiveprogrammingonios/content/images/MVVM_high_level.png)

>通过将定制的业务逻辑放置于视图模型中，我避免了视图控制器的臃肿化，视图控制器仅需要根据视图模型的协议来知道如何显示即可。 MVVM是子类化视图控制器的一个很好的选择。

VieModel 中不是只有属性, 也包含处理数据的方法

```
RAC(imageView, image) = [RACObserve(self.photoModel, fullsizeData) map:^id (id value){
	return [UIImage imageWithData:value];
}];
这样就将UIImageView的image属性和数据模型photoModel的fullsizeData属性进行了绑定
```
```
在任意的ViewModel中将相关的属性和View控件相绑定
@property (nonatomic, strong ) FRPPhotoModel *photoModel;

RAC(self.imageView, image) = [[RACObserve(self, photoModel.thumbnailData) ignore:nil]
	map:^(NSData *data){
		return [UIImage imageWithData:data];
    }];

注意看我们观察的是self的photoModel.thumbnailData的关键路径，而非self.photoModel的thumbnailData的关键路径。
```
```
[SVProgressHUD show];
//Fetch data
[[FRPPhotoImporter fetchPhotoDetails:self.photoModel]
        subscribeError:^(NSError * error){
            [SVProgressHUD showErrorWithStatus:@"Error"];
        }
        completed:^{
            [SVProgressHUD dismiss];
        }];

+ (RACReplaySubject *)fetchPhotoDetails:(FRPPhotoModel *)photoModel {
    RACReplaySubject * subject = [RACReplaySubject subject];
    NSURLRequest *request = [self photoURLRequest:photoModel];

    [NSURLConnection sendAsynchronousRequest:request
        queue:[NSOperationQueue mainQueue]
        completionHandler:^ (NSURLResponse *response, NSData * data, NSError *connectionError){
            if(data){
                id results = [NSJSONSerialization JSONObjectWithData:data options:0 error:nil][ @"photo" ];

                [self configurePhotoModel:photoModel withDictionary:results];
                [self downloadFullsizedImageForPhotoModel:photoModel];

                [subject sendNext:photoModel];
                [subject sendCompleted];
            }
            else{
                [subject sendError:connectionError];
            }
        }];

    return subject;
}
```
```
RACSignal *photoSignal = [FRPPhotoImporter importPhotos];
RACSignal *photosLoaded = [photoSignal catch:^RACSignal *(NSError *error) {
    NSLog(@"Couldn't fetch photos from 500px : %@",error);
    return [RACSignal empty];
}];
RAC(self, photosArray) = photosLoaded;
[photosLoaded subscribeCompleted: ^{
    @strongify(self);
    [self.conllectionView reloadData];
}];
比起subscribeError: 方法，catch: 方法处理的更为巧妙：它允许无错误值的信号穿透它，仅在信号有错误事件发生时才会调用它的block并发送其在发生错误时的返回值。这里我们使用catch:
方法，来过滤无错误的值。

上面的代码等效于:
RAC(self, photosArray) = [[[[FRPPhotoImporter importPhotos]
        doCompleted:^{
            @strongify(self);
            [self.collectionView reloadData];
        }] logError] catchTo:[RACSignal empty]];
```
```
+ (RACSignal *)fetchPhotoDetails:(FRPPhotoModel *)photoModel {
    NSURLRequest *request = [self photoURLRequest:photoModel];
    return [[[[[[NSURLConnection rac_sendAsynchronousRequest:request]
                                map:^id(RACTuple *value){
                                    return [value second];
                                }]
                                deliverOn:[RACScheduler mainThreadScheduler]]
                                    map:^id (NSData *data) {
                                        id results = [NSJSONSerialization JSONObjectWithData:data
                                                                       options:0 error:nil][@"photo"];
                                        [self configurePhotoModel:photoModel withDictionary:results];
                                        [self downloadFullsizedImageForPhotoModel:photoModel];
                                        return photoModel;
                                    }] publish] autoconnect];
}

publish返回一个RACMulitcastConnection,当信号连接上时，他将订阅该接收信号。autoconnect为我们做的是：当它返回的信号被订阅，连接到 该(订阅背后的)信号（underly signal）。
```
```
有一个禅宗佛教的概念叫做"初心"。禅宗法师[铃木俊隆]写道："初学者的心中有很多可能性（潜意识的点子），但在专家心里(这种可能性/点子)就相对少很多"。
```
